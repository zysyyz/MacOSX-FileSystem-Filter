# MacOSX-FileSystem-Filter
A file system filter for Mac OS X

Mac OS X doesn't support a full fledged file system filtering like Windows as Apple believes that BSD stackable file system doesn't fit Mac OS X. The available kernel authorization subsystem ( kauth ) allows only filtering open requests and a limited number of operations on file/directory. It doesn't allow filtering read and write operations and provides a limited control over file system operations as a vnode is already created at the moment of kauth callback invoking. In Windows parlance a kauth callback is a postoperation callback for create/open request ( IRP_MJ_CREATE for Windows ), that is all you have for Mac OS X. Not much really.

 The filtering can also be implemented by registering MAC ( Mandatory Access Control ) layer. But MAC has limited functionality and not consistent in relation to file system filtering as it was not designed as a file system filter layer. Instead of being called by VFS layer MAC registered callbacks are scattered through the kernel code like system calls and other kernel subsystems and if I remember correctly MAC was declared by Apple as deprecated for usage by third party developers.

 There is no official support from Apple if you need to filter read/write requests or perform some modifications on lookup or create call. To overcome this I developed a hooking technique for VFS layer that allows to perform full control over file system operations including creating an isolation file system when a filter creates vnodes instead of a file system driver thus having a full control over file system operations. I used this technique for two projects to implement filtering for lookup, create, read and write requests and to implement an isolation file system.

 FltFakeFSD.h and FltFakeFSD.cpp are optional files that helps to infer the vnode and vnodeop_desc structures layout that are not declared in SDK.  This is achieved by registering a dummy file system, creating a vnode and inspecting it to find required offsets. All you need to do is to call FltGetVnodeLayout() in a filter initialization code.

 Alternatively you can extract declarations from XNU code at opensource.apple.com. An alternative implementation without using the fake FSD can be found in VersionDependent.h and VersionDependent.cpp files, where struct vnodeop_desc_Yosemite was borrowed from Apple's open source code for Yosemite(10.10), it happened that vnode and vnodeop_desc structures haven't change in all latest kernel versions so this code works for Mavericks(10.9) and El Capitan(10.11).

 The filter registers the kauth callback FltIOKitKAuthVnodeGate::VnodeAuthorizeCallback which in turn calls FltHookVnodeVopAndParent that hooks a vnode related operations. So for example if an application opens a file FileX in a directory DirX a vnode for a DirX is provided as a parameter for VnodeAuthorizeCallback so the following calls to file system's lookup or create operations for FileX will be visible for file system filter. In case of lookup do not forget about name cache, so lookup is not always called when an application calls open( FileX ), but if you need to see all lookup operations it is possible to disable name caching for a vnode.

 The filter registers operations it wants to intercept in gFltVnodeVopHookEntries array. The skeleton filter immediately calls an original function from a hook but if you want to see a filter in action look at DldVNodeHook.cpp which is a more advanced implementation at MacOSX-Kernel-Filter that also implements an isolation filter for read and write operations on a file by redirecting read and write to a sparse file on another volume. The isolation filter is implemented at DldCoveringVnode.cpp and a sparse file at DldSparseFile.cpp

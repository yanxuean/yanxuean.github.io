---
layout: post
title: runC
categories: [CloudNative]
description: runC
keywords: runC, container
---


# runC
һ�������й��ߣ��û�����container��

## 1 һЩ��������

runc����������Ҫָ��root����������Ĭ�϶�Ϊ/run/runc��root�����������洢state�ġ�
���ͨ��containerd������container��  
```
runc --root /run/containerd/runc/k8s.io  list
```

runc  state   ���Ƿ���root/<containerid>�����state.json�ļ�
runc  list    ���Ƿ���root������Ŀ¼��Ŀ¼��Ϊcontainerid��Ȼ��Ҳ����state.json�ļ���
        ������state������state.json�е�initprocessstarttime��proc/pid/stat�е�starttime���Ƿ����exec.fifo�ļ����жϡ�
               ���������proc/pid/stat����initprocessstarttime��proc/pid/stat�е�starttime����ȣ���proc/pid/stat�е�stateΪzombie��dead����Ϊstoped̬
               �������exec.fifo�ļ���created״̬�� �������������running̬
runc exec��
      cri��exec   �ȵ���shim��exec��ֻ�ǽ�process�ṹ�����ٵ���start
           func (c *criService) execInContainer(ctx context.Context, id string, opts execOptions) 

## 2 ʹ��runC�Ĵ������̣�
1. shim����process1 ִ��runc create
2. runc create����processAִ��runc init����ͨ��pipe��runc initͬ���ҽ������� 
3. processA����processBִ��clone��unshare���л�namespaces����ǰֻ��pid��network��ipc��uts��mount���� processA���Ҫexit
4. processB����processC������namespace������Ч��pid namespaceֻ�ڶ��Ӳ���Ч���� processB���Ҫexit
5. processC�ٴ�runc create��process1���ս������ã����������л�root dir��setuid��setgid�ȶ�����Ȼ��ͣ�����ȴ�ִ��user process
6. runc start֪ͨprocessCִ��user process

## 3 ִ��Ч��
#### =============���̹�ϵ==========================
```
root      2289     1  2 Jul24 ?    /usr/bin/containerd --root /mnt/sda1/var/lib/containerd

root      2815  2289  0 Jul24 ?    containerd-shim 
-namespace k8s.io 
-workdir /mnt/sda1/var/lib/containerd/io.containerd.runtime.v1.linux/k8s.io/04440a4d0265021e4e67a62dc190d1cfa38ba9a0b2185d1ec71b14445ba0ae22 
-address /run/containerd/containerd.sock 
-containerd-binary /usr/bin/containerd

root      2823  2815  0 Jul24 ?    runc 
--root /run/containerd/runc/k8s.io 
--log /run/containerd/io.containerd.runtime.v1.linux/k8s.io/04440a4d0265021e4e67a62dc190d1cfa38ba9a0b2185d1ec71b14445ba0ae22/log.json 
--log-format json 
create 
--bundle /run/containerd/io.containerd.runtime.v1.linux/k8s.io/04440a4d0265021e4e67a62dc190d1cfa38ba9a0b2185d1ec71b14445ba0ae22 
--pid-file /run/containerd/io.containerd.runtime.v1.linux/k8s.io/04440a4d0265021e4e67a62dc190d1cfa38ba9a0b2185d1ec71b14445ba0ae22/init.pid 
--no-pivot 
04440a4d0265021e4e67a62dc190d1cfa38ba9a0b2185d1ec71b14445ba0ae22

root      2832  2823  0 Jul24 ?    runc init
```

#### ==============Ŀ¼===========================
runc��
    rootĿ¼�� ��Ҫ��state.json�ļ�
    bundle��
             config.json   init.pid   log.json   rootfs


## 4 �������

### runc ��libcontainer�Ľ���

#### 1��runc create��ʵ�֣�
```
factory = libcontainer.New(...)
container = factory.Create(containerid, libcontainer.Config)
libcontainer.Process.Init = true    ��--����Ĳ����ܴ�Ӱ��
container.Start(libcontainer.Process)       ��-- ��ʱ������һ��׮���̣�����ִ��user process.
```

#### 2������runc  xxx��ʵ��
```
factory = libcontainer.New(...)
container = factory.Load(containerid)    ��--������container
```

##### 2.1��runc start
```
container.Exec()              ��--��container��signal������ִ��user process
```
##### 2.2��runc exec         ��--��container��ִ��һ���½���
```
libcontainer.Process.Init = false    ��--����Ĳ����ܴ�Ӱ�졣����ִ�е�runc init���̲���ȴ���ֱ��ִ��user process��
container.Run(libcontainer.Process)
```

##### 2.3��runc kill
```
container.Signal(signal, isAll)
```

#### 3��runc run
```
factory = libcontainer.New(...)
container = factory.Create(containerid, libcontainer.Config)
libcontainer.Process.Init = true    ��--����Ĳ����ܴ�Ӱ��
container.Run(libcontainer.Process)     <==����container.Start + container.Exec��ִ�����û����̣��� 
                                        ��runc exec��ͬ����ִ��container.Run����process�����init�����෴
```
																  
#### container object

```
// Container is a libcontainer container object.
type Container interface {
	BaseContainer

	// Methods below here are platform specific
	Checkpoint(criuOpts *CriuOpts) error
	Restore(process *Process, criuOpts *CriuOpts) error
	Pause() error
	Resume() error
	// NotifyOOM returns a read-only channel signaling when the container receives an OOM notification.
	NotifyOOM() (<-chan struct{}, error)
	// NotifyMemoryPressure returns a read-only channel signaling when the container reaches a given pressure level
	NotifyMemoryPressure(level PressureLevel) (<-chan struct{}, error)
}
type BaseContainer interface {
    ID() string       // Returns the ID of the container
    Status() (Status, error)     // Returns the current status of the container.
    State() (*State, error)      // State returns the current container's state information.
    Config() configs.Config      // Returns the current config of the container.
    Processes() ([]int, error)   // Returns the PIDs inside this container. The PIDs are in the namespace of the calling process.
    Stats() (*Stats, error)      // Returns statistics for the container.
    Set(config configs.Config) error     // Set resources of container as configured
    Start(process *Process) (err error)  // Start a process inside the container. ��ʱ����ִ��user process.
    Exec() error   // Exec signals the container to exec the users process at the end of the init.  ��container��signal������ִ��user process
    Run(process *Process) (err error)    // Run immediately starts the process inside the container. ִ����user process
    Destroy() error                      // Destroys the container, ɱ�����н���
    Signal(s os.Signal, all bool) error  // Signal sends the provided signal code to the container's initial process.
```

### prepareRootfs��ʵ��

prepareRootfs��runc init��StartInitialization��ִ�У��Ѿ����µ�mount namespace��

#### �����ܣ�

prepareRoot
  ���RootPropagationδ�裬������/Ϊrslave������RootPropagation���ã���pod��˫�򴫲�ʱ��rootPropagation����Ϊshare��
  ������rootfs�������mountpointΪprivate����rootfs�Լ�����mountpoint����Ϊ�Լ���
  ��rootfsִ��bind������ת��Ϊmountpoint

mountToRootfs
    mount�û���mount��rootfsĿ¼�¸��Զ�Ӧ��mountpoint

pivot���л�/��
```
  chdir(rootfs)
    if no_pivot
         Mount(rootfs, "/", "", unix.MS_MOVE, "")     ��--����rootfs�Ƶ�/��
         unix.Chroot(".")     ��---�л�/ָ��rootfs
         Chdir("/")
    else if�û�������newMountNamespace       <--pivot-root
         unix.Chdir(rootfs)
         unix.PivotRoot(".", ".")
         unix.Fchdir(oldroot)        ��--Ϊ����pivot-root��ǹ��
         //��ʼumount oldroot
         unix.Mount("", ".", "", unix.MS_SLAVE|unix.MS_REC, "") <--��Ϊ����֪���������private
         unix.Unmount(".", unix.MNT_DETACH)
         unix.Chdir("/")
    else
     unix.Chroot(".")
         unix.Chdir("/")
```

##### ====================������룺=======================
```
// prepareRootfs sets up the devices, mount points, and filesystems for use
// inside a new mount namespace. It doesn't set anything as ro. You must call
// finalizeRootfs after this function to finish setting up the rootfs.
func prepareRootfs(pipe io.ReadWriter, iConfig *initConfig) (err error) {
	config := iConfig.Config
	if err := prepareRoot(config); err != nil {
		return newSystemErrorWithCause(err, "preparing rootfs")
	}

	for _, m := range config.Mounts {
		for _, precmd := range m.PremountCmds {
			if err := mountCmd(precmd); err != nil {
				return newSystemErrorWithCause(err, "running premount command")
			}
		}

                   ���û�Ҫmount��Ŀ¼mount������Ŀ��·��������rootfs��Ϊǰ׺�����ڻ���host��Ŀ¼�ṹ��mount��
		if err := mountToRootfs(m, config.Rootfs, config.MountLabel); err != nil {     
			return newSystemErrorWithCausef(err, "mounting %q to rootfs %q at %q", m.Source, config.Rootfs, m.Destination)
		}

		for _, postcmd := range m.PostmountCmds {
			if err := mountCmd(postcmd); err != nil {
				return newSystemErrorWithCause(err, "running postmount command")
			}
		}
	}

	setupDev := needsSetupDev(config)

	if setupDev {
		if err := createDevices(config); err != nil {
			return newSystemErrorWithCause(err, "creating device nodes")
		}
		if err := setupPtmx(config); err != nil {
			return newSystemErrorWithCause(err, "setting up ptmx")
		}
		if err := setupDevSymlinks(config.Rootfs); err != nil {
			return newSystemErrorWithCause(err, "setting up /dev symlinks")
		}
	}

	// Signal the parent to run the pre-start hooks.
	// The hooks are run after the mounts are setup, but before we switch to the new
	// root, so that the old root is still available in the hooks for any mount
	// manipulations.
	// Note that iConfig.Cwd is not guaranteed to exist here.
	if err := syncParentHooks(pipe); err != nil {
		return err
	}

	// The reason these operations are done here rather than in finalizeRootfs
	// is because the console-handling code gets quite sticky if we have to set
	// up the console before doing the pivot_root(2). This is because the
	// Console API has to also work with the ExecIn case, which means that the
	// API must be able to deal with being inside as well as outside the
	// container. It's just cleaner to do this here (at the expense of the
	// operation not being perfectly split).

	if err := unix.Chdir(config.Rootfs); err != nil {
		return newSystemErrorWithCausef(err, "changing dir to %q", config.Rootfs)
	}

	if config.NoPivotRoot {
		err = msMoveRoot(config.Rootfs)
	} else if config.Namespaces.Contains(configs.NEWNS) {
		err = pivotRoot(config.Rootfs)              
	} else {
		err = chroot(config.Rootfs)
	}
	if err != nil {
		return newSystemErrorWithCause(err, "jailing process inside rootfs")
	}

	if setupDev {
		if err := reOpenDevNull(); err != nil {
			return newSystemErrorWithCause(err, "reopening /dev/null inside container")
		}
	}

	if cwd := iConfig.Cwd; cwd != "" {
		// Note that spec.Process.Cwd can contain unclean value like  "../../../../foo/bar...".
		// However, we are safe to call MkDirAll directly because we are in the jail here.
		if err := os.MkdirAll(cwd, 0755); err != nil {
			return err
		}
	}

	return nil
}
```

```
func prepareRoot(config *configs.Config) error {
	flag := unix.MS_SLAVE | unix.MS_REC
	if config.RootPropagation != 0 {
		flag = config.RootPropagation     ��--�ñ�־�����runc���spec����˫��mount��������ֵ�ͻ�����Ϊrshare
	}
	if err := unix.Mount("", "/", "", uintptr(flag), ""); err != nil {      ��--�ѵ�ǰ/Ŀ¼��Ϊslave���ݹ�
		return err
	}

	��--docker1.12 runc1.0rc2��֮ǰ���˴�ֻ��no-pivotʱ��������runc #1148
         rootfs�����һ��mountpoint�����share�����Ϊprivate��rootfs�����������mountpoint������Լ�
	// Make parent mount private to make sure following bind mount does
	// not propagate in other namespaces. Also it will help with kernel
	// check pass in pivot_root. (IS_SHARED(new_mnt->mnt_parent))
	if err := rootfsParentMountPrivate(config.Rootfs); err != nil {  
		return err                                                                                         
	}

          ��rootfs��������Ҳ��Ϊmountpoint
	return unix.Mount(config.Rootfs, config.Rootfs, "bind", unix.MS_BIND|unix.MS_REC, "")  
}

```

```
// pivotRoot will call pivot_root such that rootfs becomes the new root
// filesystem, and everything else is cleaned up.
func pivotRoot(rootfs string) error {
	// While the documentation may claim otherwise, pivot_root(".", ".") is
	// actually valid. What this results in is / being the new root but
	// /proc/self/cwd being the old root. Since we can play around with the cwd
	// with pivot_root this allows us to pivot without creating directories in
	// the rootfs. Shout-outs to the LXC developers for giving us this idea.

	oldroot, err := unix.Open("/", unix.O_DIRECTORY|unix.O_RDONLY, 0)
	defer unix.Close(oldroot)

	newroot, err := unix.Open(rootfs, unix.O_DIRECTORY|unix.O_RDONLY, 0)
	defer unix.Close(newroot)

	// Change to the new root so that the pivot_root actually acts on it.
	if err := unix.Fchdir(newroot); err != nil {
		return err
	}

	if err := unix.PivotRoot(".", "."); err != nil {              <--��ʱ/Ϊnew��/proc/self/cwdΪold
		return fmt.Errorf("pivot_root %s", err)
	}

	// Currently our "." is oldroot (according to the current kernel code).
	// However, purely for safety, we will fchdir(oldroot) since there isn't
	// really any guarantee from the kernel what /proc/self/cwd will be after a
	// pivot_root(2).
	if err := unix.Fchdir(oldroot); err != nil {
		return err
	}

	// Make oldroot rslave to make sure our unmounts don't propagate to the
	// host (and thus bork the machine). We don't use rprivate because this is
	// known to cause issues due to races where we still have a reference to a
	// mount while a process in the host namespace are trying to operate on
	// something they think has no mounts (devicemapper in particular).
	if err := unix.Mount("", ".", "", unix.MS_SLAVE|unix.MS_REC, ""); err != nil {
		return err
	}
	// Preform the unmount. MNT_DETACH allows us to unmount /proc/self/cwd.
	if err := unix.Unmount(".", unix.MNT_DETACH); err != nil {
		return err
	}

	// Switch back to our shiny new root.
	if err := unix.Chdir("/"); err != nil {
		return fmt.Errorf("chdir / %s", err)
	}
	return nil
}

```



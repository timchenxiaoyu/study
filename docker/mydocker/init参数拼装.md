通过自身的init启动，先组装如下参数
```go
	initCmd, err := os.Readlink("/proc/self/exe")
	if err != nil {
		log.Errorf("get init process error %v", err)
		return nil, nil
	}

	cmd := exec.Command(initCmd, "init")
	设定启动的namespace
	cmd.SysProcAttr = &syscall.SysProcAttr{
		Cloneflags: syscall.CLONE_NEWUTS | syscall.CLONE_NEWPID | syscall.CLONE_NEWNS |
			syscall.CLONE_NEWNET | syscall.CLONE_NEWIPC,
	}
	cmd.ExtraFiles = []*os.File{readPipe}
    cmd.Env = append(os.Environ(), envSlice...)
    NewWorkSpace(volume, imageName, containerName)
    cmd.Dir = fmt.Sprintf(MntUrl, containerName)

```
上面是拼装init所需的启动参数和启动方式
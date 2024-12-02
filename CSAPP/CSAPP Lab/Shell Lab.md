# 前言

**实验概述**:

- 通过编写一个支持作业控制的简单 Unix shell 程序，熟悉进程控制和信号相关概念。

**具体函数实现**:

- **eval**：解析和解释命令行的主例程。
- **builtin_cmd**：识别和解释内置命令（quit、fg、bg、jobs）。
- **do_bgfg**：实现 bg 和 fg 内置命令。
- **waitfg**：等待前台作业完成。
- **sigchld_handler**：捕获 SIGCHILD 信号。
- **sigint_handler**：捕获 SIGINT（ctrl - c）信号。
- **sigtstp_handler**：捕获 SIGTSTP（ctrl - z）信号。

**实验提示**

1. 仔细阅读第 8 章（异常控制流）。
2. 利用跟踪文件指导 shell 开发，从 trace01.txt 开始，确保输出与参考 shell 相同，再逐步测试其他跟踪文件。
3. 函数`waitpid`、`kill`、`fork`、`execve`、`setpgid`和`sigprocmask`很有用，`waitpid`的`WUNTRACED`和`WNOHANG`选项也有用。
4. 实现信号处理程序时，使用`kill`函数发送信号给整个前台进程组时用`-pid`代替`pid`。
5. 建议在`waitfg`中用`sleep`函数的循环，在`sigchld_handler`中仅调用一次`waitpid`来处理收割工作。
6. 在`eval`中，父进程在`fork`前用`sigprocmask`阻塞`SIGCHLD`信号，添加子进程到作业列表后再解除阻塞，子进程在`exec`新程序前也要解除阻塞。
7. 避免运行如`more`、`less`、`vi`和`emacs`等会影响终端设置的程序，使用简单文本程序如`/bin/ls`、`/bin/ps`和`/bin/echo`。
8. shell 从标准 Unix shell 运行时在前台进程组，创建子进程后应让子进程调用`setpgid(0, 0)`进入新进程组，确保前台只有 shell，shell 收到 SIGINT 后应转发给合适的前台作业（进程组）。

```C
/*
 * main - The shell's main routine 
 */
int main(int argc, char **argv) 
{
    char c;
    char cmdline[MAXLINE];
    int emit_prompt = 1; /* emit prompt (default) */

    /* Redirect stderr to stdout (so that driver will get all output
     * on the pipe connected to stdout) */
    dup2(1, 2);

    /* Parse the command line */
    while ((c = getopt(argc, argv, "hvp")) != EOF) {
        switch (c) {
        case 'h':             /* print help message */
            usage();
	    break;
        case 'v':             /* emit additional diagnostic info */
            verbose = 1;
	    break;
        case 'p':             /* don't print a prompt */
            emit_prompt = 0;  /* handy for automatic testing */
	    break;
	default:
            usage();
	}
    }

    /* Install the signal handlers */

    /* These are the ones you will need to implement */
    Signal(SIGINT,  sigint_handler);   /* ctrl-c */
    Signal(SIGTSTP, sigtstp_handler);  /* ctrl-z */
    Signal(SIGCHLD, sigchld_handler);  /* Terminated or stopped child */

    /* This one provides a clean way to kill the shell */
    Signal(SIGQUIT, sigquit_handler); 

    /* Initialize the job list */
    initjobs(jobs);

    /* Execute the shell's read/eval loop */
    while (1) {

	/* Read command line */
	if (emit_prompt) {
	    printf("%s", prompt);
	    fflush(stdout);
	}
	if ((fgets(cmdline, MAXLINE, stdin) == NULL) && ferror(stdin))
	    app_error("fgets error");
	if (feof(stdin)) { /* End of file (ctrl-d) */
	    fflush(stdout);
	    exit(0);
	}

	/* Evaluate the command line */
	eval(cmdline);
	fflush(stdout);
	fflush(stdout);
    } 

    exit(0); /* control never reaches here */
}
```

# eval

如果用户请求了一个内置命令（如 quit、jobs、bg 或 fg），那么要立即执行它。否则，就创建一个子进程，并在子进程的环境中运行该任务。如果该任务是在前台运行，那就等待它终止，然后返回。注意：每个子进程都必须有一个唯一的进程组 ID，这样当我们在键盘上输入 Ctrl - C（对应 SIGINT）或 Ctrl - Z（对应 SIGTSTP）时，我们的后台子进程就不会从内核接收到这些信号。

```C
/* 
 * eval - Evaluate the command line that the user has just typed in
 * 
 * If the user has requested a built-in command (quit, jobs, bg or fg)
 * then execute it immediately. Otherwise, fork a child process and
 * run the job in the context of the child. If the job is running in
 * the foreground, wait for it to terminate and then return.  Note:
 * each child process must have a unique process group ID so that our
 * background children don't receive SIGINT (SIGTSTP) from the kernel
 * when we type ctrl-c (ctrl-z) at the keyboard.  
*/
void eval(char *cmdline) 
{
    char* argv[MAXARGS];
    int bg;
    pid_t pid;
    sigset_t mask;
    bg = parseline(cmdline, argv);
    if (!builtin_cmd(argv)) {
        sigemptyset(&mask);
        sigaddset(&mask, SIGCHLD);
        sigprocmask(SIG_BLOCK, &mask, NULL);
        if ((pid = fork()) == 0) { // child
            sigprocmask(SIG_UNBLOCK, &mask, NULL);

            if (execvp(argv[0], argv) < 0) {
                printf("%s: Command not found\n", argv[0]);
                exit(1); 
            }

        } else if (pid > 0) { // parent
            // 添加任务
            if (!bg)
                addjob(jobs, pid, FG, cmdline);
            else
                addjob(jobs, pid, BG, cmdline);
            // 解除屏蔽
            sigprocmask(SIG_UNBLOCK, &mask, NULL);
            
            if (!bg)
                waitfg(pid); 
            else
                printf("[%d] (%d) %s", pid2jid(pid), pid, cmdline);
        } else {
            unix_error("forking error");
            return;
        }
    }
    return;
}
```

如果子进程在父进程能够开始运行前就结束了，那么`addjob`函数以及`sigchld_hanlder`中的`deletejob`函数会以错误的方式调用。所以要通过在调用frok之前，阻塞SIGCHLD信号，然后在调用addjob之后取消阻塞这些信号，来保证在子进程被添加到作业列表之后回收该子进程。注意，子进程继承了它们父进程的被阻塞集合，所以我们必须在调用 execve 之前，小心地解除子进程中阻塞的 SIGCHLD 信号。详见书上**8.5.6**。

# builtin_cmd

如果是一个内置命令，就执行它。

```C
/* 
 * builtin_cmd - If the user has typed a built-in command then execute
 *    it immediately.  
 */
int builtin_cmd(char **argv) 
{
    if (!strcmp(argv[0], "quit")) {
        exit(0);
    } else if (!strcmp(argv[0], "&")) {
        return 1;
    } else if (!strcmp(argv[0], "jobs")) {
        listjobs(jobs);
        return 1;
    } else if (!strcmp(argv[0], "bg") || !strcmp(argv[0], "fg")) {
        // bg or fg
        do_bgfg(argv);
        return 1;
    }
    return 0;     /* not a builtin command */
}
```

对四种对应的内置命令进行处理。

# do_bgfg

执行内置的 `bg`（后台运行）和 `fg`（前台运行）命令。

```C
/* 
 * do_bgfg - Execute the builtin bg and fg commands
 */
void do_bgfg(char **argv) 
{
    struct job_t* job;
    char* tmp = argv[1];
    int jid;
    pid_t pid;

    // 无id
    if (tmp == NULL) {
        printf("%s command requires PID or %%jobid argument\n", argv[0]);
        return;
    }

    if (tmp[0] == '%') {
        // jid
        jid = atoi(tmp + 1);
        job = getjobjid(jobs, jid);
        if (job == NULL) {
            printf("%s: No such job\n", tmp);  
            return;  
        } else {
            pid = job->pid;
        }
    } else if (isdigit(tmp)) {
        // pid
        pid = atoi(tmp);
        job = getjobpid(jobs, pid);
        if (job == NULL) {  
            printf("(%d): No such process\n", pid);  
            return;  
        }
    } else {
        printf("%s: argument must be a PID or %%jobid\n", argv[0]);
        return;
    }
    kill(-pid, SIGCONT);
    if (!strcmp(argv[0], "fg")) {
        job->state = FG;
        waitfg(pid);
    } else {
        printf("[%d] (%d) %s", job->jid, job->pid, job->cmdline);
        job->state = BG;
    }
    return;
}
```

# waitfg

阻塞当前进程，直到进程标识符（pid）所对应的进程不再是前台进程为止。

```C
/* 
 * waitfg - Block until process pid is no longer the foreground process
 */
void waitfg(pid_t pid)
{
    struct job_t* job;
    job = getjobpid(jobs, pid); 
    if (job)
        while (pid == fgpid(jobs))
            usleep(100);
    return;
}
```

# sigchld_handler

每当一个子作业终止（变成僵死进程），或者因为接收到了 SIGSTOP 或 SIGTSTP 信号而停止时，内核就会向 shell 发送一个 SIGCHLD 信号。回收所有可用的僵尸子进程，但不会等待任何其他正在运行的子进程终止。

```C
/* 
 * sigchld_handler - The kernel sends a SIGCHLD to the shell whenever
 *     a child job terminates (becomes a zombie), or stops because it
 *     received a SIGSTOP or SIGTSTP signal. The handler reaps all
 *     available zombie children, but doesn't wait for any other
 *     currently running children to terminate.  
 */
void sigchld_handler(int sig) 
{
    int olderrno = errno;
    pid_t pid;
    int status;
    struct job_t* job;
    sigset_t mask_all, prev_all;
    sigfillset(&mask_all);

    // 回收子进程
    while ((pid = waitpid(fgpid(jobs), &status, WNOHANG | WUNTRACED)) > 0) {
        job = getjobpid(jobs, pid);
        if (WIFEXITED(status)) { 
            // 正常退出，删除并返回
            // 阻塞信号，保护全局数据结构，详见8.5.6
            sigprocmask(SIG_SETMASK, &mask_all, &prev_all); 
            deletejob(jobs, pid);
            sigprocmask(SIG_SETMASK, &prev_all, NULL);
        } else if (WIFSIGNALED(status)) {
            // 由信号终止，打印信息，删除
            printf("Job [%d] (%d) terminated by signal %d\n", job->jid, pid, WTERMSIG(status));
            sigprocmask(SIG_SETMASK, &mask_all, &prev_all);
            deletejob(jobs, pid);
            sigprocmask(SIG_SETMASK, &prev_all, NULL);
        }else if (WIFSTOPPED(status)) {
            // 由信号停止，更改状态
            printf("Job [%d] (%d) stopped by signal %d\n", job->jid, pid, WSTOPSIG(status));
            sigprocmask(SIG_SETMASK, &mask_all, &prev_all);
            job->state = ST;
            sigprocmask(SIG_SETMASK, &prev_all, NULL);
        }
    }

    if (pid < 0 && errno != ECHILD)
        unix_error("waitpid fail");
    errno = olderrno;

    return;
}
```

# sigint_handler

在键盘上按下 Ctrl - C 时，内核就会向 shell 发送一个 SIGINT 信号。捕获该信号并将其发送给前台作业。

```C
/* 
 * sigint_handler - The kernel sends a SIGINT to the shell whenver the
 *    user types ctrl-c at the keyboard.  Catch it and send it along
 *    to the foreground job.  
 */
void sigint_handler(int sig) 
{
    int olderrno = errno;
    pid_t pid = fgpid(jobs); // 得到前台进程id
    if (pid)
        kill(-pid, sig);
    errno = olderrno;
    return;
}
```

# sigstp_handler

在键盘上按下 Ctrl - Z 时，内核就会向 shell 发送一个 SIGTSTP 信号。捕获这个信号，然后通过向前台作业发送一个 SIGTSTP 信号来暂停它。

```C
/*
 * sigtstp_handler - The kernel sends a SIGTSTP to the shell whenever
 *     the user types ctrl-z at the keyboard. Catch it and suspend the
 *     foreground job by sending it a SIGTSTP.  
 */
void sigtstp_handler(int sig) 
{
    int olderrno = errno;
    pid_t pid = fgpid(jobs);
    if (pid)
        kill(-pid, sig);
    errno = olderrno;
    return;
}
```


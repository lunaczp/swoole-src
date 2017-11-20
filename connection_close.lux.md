# Connection Close

调用栈
===
(from lldb bt)
```
  * frame #0: 0x000000010f8e7e23 swoole.so`php_swoole_onClose(serv=0x00007f8c7600bc00, info=0x00007fff51711904) at swoole_server.c:1155
    frame #1: 0x000000010f9118f1 swoole.so`swFactory_end(factory=<unavailable>, fd=1) at Factory.c:131
    frame #2: 0x000000010f922958 swoole.so`swWorker_onTask(factory=<unavailable>, task=0x00007fff51711994) at Worker.c:254
    frame #3: 0x000000010f91fd43 swoole.so`swReactorProcess_onClose(reactor=<unavailable>, event=<unavailable>) at ReactorProcess.c:404
    frame #4: 0x000000010f91de05 swoole.so`swReactorThread_onClose(reactor=0x00007f8c7600c208, event=<unavailable>) at ReactorThread.c:371
    frame #5: 0x000000010f924707 swoole.so`swPort_onRead_raw(reactor=0x00007f8c7600c208, port=<unavailable>, event=0x00007fff51712260) at Port.c:216
    frame #6: 0x000000010f915423 swoole.so`swReactorKqueue_wait(reactor=0x00007f8c7600c208, timeo=<unavailable>) at ReactorKqueue.c:329
    frame #7: 0x000000010f9202e6 swoole.so`swReactorProcess_loop(pool=<unavailable>, worker=0x00007fff51712320) at ReactorProcess.c:377
    frame #8: 0x000000010f920ab4 swoole.so`swReactorProcess_start(serv=0x00007f8c7600bc00) at ReactorProcess.c:112
    frame #9: 0x000000010f918868 swoole.so`swServer_start(serv=0x00007f8c7600bc00) at Server.c:673
    frame #10: 0x000000010f8e9769 swoole.so`zim_swoole_server_start(ht=<unavailable>, return_value=0x000000010f562218, return_value_ptr=<unavailable>, this_ptr=<unavailable>, return_value_used=<unavailable>) at swoole_server.c:1770
    frame #11: 0x000000010e87bc1d php`dtrace_execute_internal + 126
    frame #12: 0x000000010f6bc978 xdebug.so`xdebug_execute_internal + 425
    frame #13: 0x000000010e8f0e19 php`zend_do_fcall_common_helper_SPEC + 1249
    frame #14: 0x000000010e8b0962 php`execute_ex + 918
    frame #15: 0x000000010e87bb47 php`dtrace_execute_ex + 223
    frame #16: 0x000000010f6bc637 xdebug.so`xdebug_execute_ex + 2426
    frame #17: 0x000000010e889dc2 php`zend_execute_scripts + 444
    frame #18: 0x000000010e8332b2 php`php_execute_script + 633
    frame #19: 0x000000010e90e76a php`do_cli + 3883
    frame #20: 0x000000010e90d6cb php`main + 1138
```

注意：`php_swoole_onClose`是作为回调被调用的,对应下面的`onClose`
```
# Factory.c:126
        if (serv->onClose != NULL)
        {
            info.fd = fd;
            info.from_id =  conn->from_id;
            info.from_fd =  conn->from_fd;
            serv->onClose(serv, &info);
        }

```

`swFactory_end`随后会调用`swReactorThread_close`，它再调用`swReactor_close`，并在这里调用`close`关闭连接句柄。  
`close`会发出`FIN,ACK`，并在收到`ACK`后最终终止TCP连接。


## todo 
- 当client close后，始终没有收到server `FIN,ACK`，client会等多久退出
- 当server延时很多很久之后才发送`FIN,ACK`，是否还能收到`ACK`（对应第一个问题）


# Stop or Kill a Thread

It is generally a bad pattern to kill a thread abruptly, in Python, and in any language.

The nice way of handling this, if you are managing your own thread, 
is to have an exit flag that each thread checks on a regular interval to see if it is time to exit:

```python
import threading


class StoppableThread(threading.Thread):
    def __init__(self,  *args, **kwargs):
        super(StoppableThread, self).__init__(*args, **kwargs)
        self._stop_event = threading.Event()

    def stop(self):
        self._stop_event.set()

    def _stopped(self):
        return self._stop_event.is_set()

    def run(self):
        while True:
            if self._stopped():
                break
```

However, when you really need to kill a thread. An example is when you are performing a long-blocking operation, and you want to interrupt it:

```python
import ctypes
import threading


def stop_thread(t: threading.Thread, exited_ok: bool = False):
    if any((t.is_alive(), exited_ok)) is False:
        raise threading.ThreadError('The thread is not active')
    
    tid, exc = ctypes.c_long(t.ident), ctypes.py_object(SystemExit)
    res = ctypes.pythonapi.PyThreadState_SetAsyncExc(tid, exc)
    
    if res == 0:
        raise ValueError('Invalid thread id')
    elif res != 1:
        ctypes.pythonapi.PyThreadState_SetAsyncExc(tid, None)
        raise SystemError("PyThreadState_SetAsyncExc failed")
```

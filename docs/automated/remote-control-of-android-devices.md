# Remote Control of Android Devices

Sometimes, we want to remotely control android devices which connected to another node like below:

``` mermaid
graph LR
  B[Emulator] --> A[Node IP:<192.168.98.87>];
  C[Android Phone] --> A;
  D[Rack] --> A;
  E[...] --> A;
```

First of all, we need to know that each adb device has a **control port**. 

For example, when you run an android emulator named `emulator-5554`, you can connect to it by running `adb connect 127.0.0.1:5555`.

So as long as we can access the port `192.168.98.87:5555`, we can remotely control `emulator-5554`.

Nginx is a good choice to do this. To do this:

Install nginx on the node.

```shell
sudo apt install nginx -y
```
    
Configure port forwarding in nginx config file:

```title="/etc/nginx/nginx.conf"
stream {
    server {
        listen 9887;
        proxy_pass 127.0.0.1:5555;
    }
}
```

Reload nginx:

```shell
sudo nginx -s reload
```

After the above steps, we can connect to the emulator by running `adb connect 192.168.98.87:9887`. 
And we can use `scrcpy` to do a screencast of the emulator.

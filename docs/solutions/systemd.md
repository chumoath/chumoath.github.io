# systemd

## 1、socat TCP 转发 DBus

- socat转发unix到tcp：`socat TCP-LISTEN:12345,fork UNIX-CONNECT:/var/run/dbus/system_bus_socket &`
- systemd配置修改：`<auth>ANONYMOUS</auth> <allow_anonymous/>`添加到`/usr/share/dbus-1/system.conf`的`<auth>EXTERNAL</auth>`前面，`systemctl restart dbus`
- d-feet: `apt install d-feet`
- d-feet连接：`Connect to other bus -> tcp:host=10.21.2.184,port=12345`

## 2、d-feet参数

- `GLib.Variant("b", False)`
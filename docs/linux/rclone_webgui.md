# Rclone Web GUI as a systemd Service

Rclone ships with a built-in web GUI. Instead of starting it by hand every time, run it as a **systemd user service** that starts automatically at login. Then you only need to open the page in the browser.

## Part 1: Create the Service

1. **Check where rclone is installed** (the service needs the full path):

    ```
    which rclone
    ```

2. **Create the service file:**

    ```
    mkdir -p ~/.config/systemd/user
    nano ~/.config/systemd/user/rclone-webgui.service
    ```

3. **Add Content:**

    ```
    [Unit]
    Description=Rclone Web GUI
    After=network-online.target

    [Service]
    ExecStart=/usr/bin/rclone rcd --rc-web-gui --rc-web-gui-no-open-browser --rc-addr 127.0.0.1:5572 --rc-user admin --rc-pass YOUR_PASSWORD
    Restart=on-failure

    [Install]
    WantedBy=default.target
    ```

    > **Remember:** Replace `YOUR_PASSWORD` with your own password, and fix the path of `rclone` if `which rclone` shows a different one.

4. **Restrict access to the file**, since it contains the password:

    ```
    chmod 600 ~/.config/systemd/user/rclone-webgui.service
    ```

---

## Part 2: Enable and Start

```
systemctl --user daemon-reload
systemctl --user enable --now rclone-webgui.service
```

Open `http://localhost:5572` and log in with the username and password from the service file. The GUI reads the remotes from the usual `~/.config/rclone/rclone.conf`.

---

## Notes

- `--rc-addr 127.0.0.1:5572` makes the GUI reachable **only from the local machine**. Do not bind it to `0.0.0.0` unless you know what you are doing, since the GUI gives access to all your cloud remotes.
- `--rc-web-gui-no-open-browser` stops rclone from opening a browser tab every time the service starts.
- **Authentication:** it is possible to remove the login with `--rc-no-auth`, but then any local process (or a malicious website, in theory) can talk to the GUI. A password is kept as a second layer of defence, even on an encrypted single-user machine.
- **Change the password:** edit the value after `--rc-pass`, then run:

    ```
    systemctl --user daemon-reload
    systemctl --user restart rclone-webgui
    ```

- **Check the logs:**

    ```
    journalctl --user -u rclone-webgui -e
    ```

- **Check the status or stop the service:**

    ```
    systemctl --user status rclone-webgui
    systemctl --user disable --now rclone-webgui
    ```

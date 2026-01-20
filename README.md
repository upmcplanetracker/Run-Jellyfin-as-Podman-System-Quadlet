Jellyfin Podman Quadlet Deployment
==================================

**Disclaimer:** I am not affiliated with, associated, authorized, endorsed by, or in any way officially connected with the Jellyfin project. This should be a drop-in replacement, but back up any config files just in case.

This guide demonstrates how to turn a Jellyfin Docker container into a **System Podman Quadlet**. Unlike standard containers, a system quadlet is managed by `systemd` and will start automatically every time your Ubuntu system boots.  I have not been successful getting this to run rootless and still have access to the QSV/GPU.  If you are running this without a GPU you can probably convert this to rootless with little effort.

System Information
------------------

*   **Tested Environment:** Ubuntu 25.10, Podman version 5.4.x
*   **Configuration Type:** System-wide Quadlet (Root)

1\. The Quadlet File (jellyfin.container)
-----------------------------------------

Download the `jellyfin.container` quadlet file to your machine. Make sure that it is owned by `root`.

2\. Installation & Setup
------------------------

### Step A: Clean up existing containers

If you are currently running Jellyfin in Docker or a manual Podman container, stop it first.


### Step B: Set Permissions

Add your user to the `video` and `render` groups to allow hardware access:

`sudo usermod -aG video,render $USER`

### Step C: Identify User IDs

Check your current User ID (UID) and Group ID (GID). Update these values in the `.container` file if they are not 1000.

`id`

### Step D: Move and Edit

Move the file to the systemd directory and edit your paths, timezone, and QSV settings:

`sudo mkdir -p /etc/containers/systemd`
`sudo mv jellyfin.container /etc/containers/systemd/`
`sudo nano /etc/containers/systemd/jellyfin.container`
    

3\. Activation
--------------

Run these commands to trigger the Quadlet generator and start your server:

`sudo systemctl daemon-reload`
`sudo systemctl start jellyfin`
    

4\. Post-Install Verification
-----------------------------

### Check GPU Access

To ensure Jellyfin can actually see your Intel Graphics for QuickSync, run:

`podman exec -it jellyfin ls -l /dev/dri`

You should see `card0` and `renderD128` in the output and not see `nobody` as a user or group.

### Check Logs

If the container isn't starting, use journalctl to see why:

`sudo journalctl -u jellyfin -f`
or
`sudo podman logs jellyfin`

5\. Updating the Container
--------------------------

To update the container (and all System Quadlets):

`sudo podman auto-update`

_Note: Because this is a system quadlet, it will now start automatically on every system reboot._

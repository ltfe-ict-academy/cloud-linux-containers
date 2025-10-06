# LXC Integration with Python

- [python3-lxc](https://github.com/lxc/python3-lxc)
- [Command line tools](https://linuxcontainers.org/lxc/documentation/#python)
- [Stéphane Graber's website](https://stgraber.org/2014/02/05/lxc-1-0-scripting-with-the-api/)


Most of the LXC functionality can now be accessed through an API exported by liblxc for which bindings are available in several languages, including Python, lua, ruby, and go.

## Python bindings 

The python bindings are typically very close to the C API except for the part where it exports proper objects instead of structs. The binding is made in two parts, the raw "_lxc" C extension and the "lxc" python overlay which provides an improved user experience.


## Installation and setup

- Update and upgrade the system: `sudo apt update && sudo apt upgrade -y`
- Install the LXC development libraries on Ubuntu with the following command: `sudo apt install -y liblxc-dev`
- Install the gcc compiler: `sudo apt install -y build-essential`
- Install `uv` library: `curl -LsSf https://astral.sh/uv/install.sh | sh`
- Run: `source $HOME/.local/bin/env`
- Create a new folder: `mkdir -p ~/linux-containers-dev && cd ~/linux-containers-dev`
- Create a new uv Python environment: `uv python install 3.10` and `uv venv --python 3.10`
- Activate the uv Python environment: `source .venv/bin/activate`
- Install the lxc python bindings: `uv pip install lxc`

> To run privileged containers you can create a virtual environment with root user.

Run the Python interpreter and import the lxc module:

```python
>> python
>> import lxc
>> exit()
```

## Build and start a container

- Create a new script: `nano create_container.py`
- Add the following code:
```python
import sys
import time

import lxc


def create_container(
    name: str,
    template_data: dict = {"dist": "ubuntu", "release": "jammy", "arch": "amd64"},
) -> None:
    """Create a container."""
    # Setup the container object
    c = lxc.Container(name)
    if c.defined:
        print("Container already exists", file=sys.stderr)
        sys.exit(1)

    # Create the container rootfs
    if not c.create(
        "download",
        lxc.LXC_CREATE_QUIET,
        template_data,
    ):
        print("Failed to create the container rootfs", file=sys.stderr)
        sys.exit(1)

    # Start the container
    if not c.start():
        print("Failed to start the container", file=sys.stderr)
        sys.exit(1)

    # Wait for the container to start
    time.sleep(5)

    if c.running:
        print("Container is started and running successfully.")
    else:
        print("Container is not running", file=sys.stderr)


if __name__ == "__main__":
    container_name = input("Enter the container name: ")
    create_container(container_name)
```
- Run the script: `systemd-run --user --scope -p Delegate=yes -- python create_container.py`
- Check the status of the container: `lxc-ls --fancy`

## Updating all running containers

- Create a new script: `nano update_all_containers.py`
- Add the following code:
```python
import lxc

for container in lxc.list_containers(as_object=True):
    print("Updating container %s" % container.name)
    # Start the container (if not started)
    started = False
    if not container.running:
        if not container.start():
            continue
        print("> Started container %s for update" % container.name)
        started = True

    if container.state != "RUNNING":
        continue

    # Wait for connectivity
    if not container.get_ips(timeout=30):
        continue

    # Run the updates
    container.attach_wait(lxc.attach_run_command, ["apt", "update"])
    container.attach_wait(lxc.attach_run_command, ["apt", "upgrade", "-y"])

    # Shutdown the container
    if started and not container.shutdown(30):
        container.stop()

    print("Done updating container %s" % container.name)
    print("-------------------------------------")
```

- Run the script: `systemd-run --user --scope -p Delegate=yes -- python update_all_containers.py`
- Check the status of the container: `lxc-ls --fancy`

## Stop and destroy all containers

- Create a new script: `nano destroy_all_containers.py`
- Add the following code:
```python
import sys

import lxc

for container in lxc.list_containers(as_object=True):
    if container.running:
        # Stop the container
        if not container.stop():
            print(f"Failed to stop the container: {container.name}", file=sys.stderr)
            sys.exit(1)
        else:
            print(f"Stopped the container: {container.name}")

    # Destroy the container
    if not container.destroy():
        print(f"Failed to destroy the container: {container.name}", file=sys.stderr)
        sys.exit(1)
    else:
        print(f"Destroyed the container: {container.name}")

    print("-------------------------------------")
```

- Run the script: `systemd-run --user --scope -p Delegate=yes -- python destroy_all_containers.py`
- Check the status of the container: `lxc-ls --fancy`

## Run a Python script inside a container

The `attach_wait` command lets you run a standard python function in the container’s namespaces.

- Create a new script: `nano run_python_script.py`
- Add the following code:
```python
import sys

import lxc

container_name = input("Enter the name of the container to run the function on: ")


c = lxc.Container(container_name)
if not c.running:
    c.start()


def print_hostname():
    with open("/etc/hostname") as fd:
        print("Hostname: %s" % fd.read().strip())


# First run on the host
print_hostname()

# Then on the container
c.attach_wait(print_hostname)

if not c.shutdown(30):
    c.stop()

# Destroy the container
if not c.destroy():
    print(f"Failed to destroy the container: {c.name}", file=sys.stderr)
    sys.exit(1)
```

- Create a new container: `systemd-run --user --scope -p Delegate=yes -- python create_container.py`
- Run the script: `systemd-run --user --scope -p Delegate=yes -- python run_python_script.py`
- Check the status of the container: `lxc-ls --fancy`
- Exit the virtual environment: `deactivate`
- Remove the virtual environment: `rm -rf .venv`
- Remove the linux-containers-dev folder: `cd .. && rm -rf linux-containers-dev`

## **Project 1: Secure SSH Connection Between Two Docker Containers**

This guide walks you through setting up a secure shell (SSH) connection from one Ubuntu container (the client) to another (the server).

### **Objective**

To create two separate Ubuntu containers that can communicate securely with each other using SSH.

### **Step-by-Step Guide**

#### **Step 1: Prepare the Ubuntu Image**

First, make sure you have the official Ubuntu image from Docker Hub.

```bash
docker pull ubuntu
```

-----

#### **Step 2: Set Up Container 1 (The SSH Server サーバー)**

This container will accept incoming SSH connections.

1.  **Run the container in interactive mode:**

    ```bash
    docker run -it --name container-1 ubuntu
    ```

    *You are now inside `container-1`.*

2.  **Update and install the SSH server:**

    ```bash
    # Update package lists
    apt-get update

    # Install the OpenSSH Server software
    apt-get install openssh-server
    ```

3.  **Configure the SSH server to allow root login:**

      * First, install a text editor like `nano`:
        ```bash
        apt-get install nano
        ```
      * Open the configuration file:
        ```bash
        nano /etc/ssh/sshd_config
        ```
      * Find the line `#PermitRootLogin prohibit-password`. Uncomment it (remove the `#`) and change `prohibit-password` to `yes`. It should look like this:
        ```
        PermitRootLogin yes
        ```
      * Save the file by pressing `Ctrl+S` and exit with `Ctrl+X`.

4.  **Set a password for the `root` user:**

    ```bash
    passwd root
    ```

    *Enter a simple password when prompted, like `123`, and confirm it.*

5.  **Start the SSH service:**

    ```bash
    service ssh start
    ```

6.  **Exit the container:**

    ```bash
    exit
    ```

-----

#### **Step 3: Set Up Container 2 (The SSH Client クライアント)**

This container will initiate the SSH connection.

1.  **Run the second container in interactive mode:**

    ```bash
    docker run -it --name container-2 ubuntu
    ```

    *You are now inside `container-2`.*

2.  **Update and install the SSH client:**

    ```bash
    # Update package lists
    apt-get update

    # Install the OpenSSH Client software
    apt-get install openssh-client
    ```

3.  **Exit the container:**

    ```bash
    exit
    ```

-----

#### **Step 4: Connect the Two Containers**

Now, we'll connect from `container-2` to `container-1`.

1.  **Find the IP Address of `container-1`:**

      * Run this command in your main terminal (not inside a container):
        ```bash
        docker inspect container-1 | grep IPAddress
        ```
      * Note down the IP address it shows (e.g., `172.17.0.2`).

2.  **⚠️ Start the containers (Very Important\!):**

      * When you `exit` an interactive container, it stops. You must restart them before trying to `exec` into them or connect to them.
        ```bash
        docker start container-1
        docker start container-2
        ```

3.  **Enter `container-2` to initiate the connection:**

    ```bash
    docker exec -it container-2 bash
    ```

4.  **Connect to `container-1` using SSH:**

      * From inside `container-2`, run the SSH command with the IP address you found.
        ```bash
        # Replace <IP_ADDRESS> with the actual IP of container-1
        ssh root@<IP_ADDRESS>
        ```
      * You will be asked if you want to continue connecting. Type `yes` and press Enter.
      * Enter the password you set for `container-1` (e.g., `123`).

✅ **Success\!** You are now logged into `container-1` from `container-2`.

-----

### **Notes on OpenSSH**

#### **`openssh-server`**

  * **Purpose:** This package turns a machine (or a container) into a **host** or **server**.
  * **Function:** It actively **listens** for incoming connection requests on a specific network port (port 22 by default).
  * **Analogy:** Think of `openssh-server` as installing a secure, locked door on a house. It allows people with the right key (credentials) to enter. **You install this on the machine you want to connect *to*.**

#### **`openssh-client`**

  * **Purpose:** This package gives a machine (or a container) the ability to act as a **client**.
  * **Function:** It provides the necessary tools, like the `ssh` command, to **initiate** a connection to an SSH server. It doesn't listen for connections itself.
  * **Analogy:** Think of `openssh-client` as having the key and knowing how to use it to open the locked door. **You install this on the machine you want to connect *from*.**

### **⚠️ Crucial Reminder: `docker start` vs. `docker exec`**

When you run a container with `docker run -it` and then type `exit`, **the container stops running**.

  * `docker exec` is used to run a command **inside an already running container**.
  * If you try to `docker exec` into a stopped container, you will get an error.

Therefore, the correct workflow after you have exited your containers is:

1.  **Start the container:** `docker start <container_name>`
2.  **Execute a command inside it:** `docker exec -it <container_name> bash`

**Always remember to `start` your container before you `exec` into it\!**

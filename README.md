---

# Setting Up Ngrok for SSH Access on Ubuntu (Free Plan)

This guide will walk you through installing and configuring ngrok so that you can expose your board's SSH port (22) to the public. Note that on the free plan, the public TCP address is ephemeral (it may change every time ngrok starts). You'll use a simple terminal command to query the current address and pass it to your friend via a JTAG connection.

---

## Step 1: Install ngrok

1. **Download the Binary:**

   Visit the [ngrok download page](https://ngrok.com/download) and download the correct binary for your Ubuntu board. For example, if your board is x86_64 or ARM, choose the corresponding version.

2. **Extract and Move ngrok:**

   Open a terminal and run the following commands (adjust the filename if necessary):

   ```bash
   unzip /path/to/ngrok-stable-linux-<arch>.zip
   sudo mv ngrok /usr/local/bin/
   ```

3. **Make Sure ngrok is Executable:**

   ```bash
   sudo chmod +x /usr/local/bin/ngrok
   ```

---

## Step 2: (Optional) Configure Authtoken

While you can run ngrok without an authtoken, it’s recommended to register for a free ngrok account so you can avoid potential connection limits and have access to additional features.

1. **Sign Up for a Free Account:**  
   Go to [ngrok.com](https://ngrok.com) and sign up.

2. **Get Your Authtoken:**  
   After registering, find your authtoken on the dashboard.

3. **Set the Authtoken:**

   ```bash
   ngrok authtoken YOUR_AUTHTOKEN
   ```

   > **Note:** Even though you don’t need a fixed address (reserved domain) on the free plan, using an authtoken helps ensure smoother operation.

---

## Step 3: Start an ngrok TCP Tunnel for SSH

Run ngrok to expose port 22 (SSH):

```bash
ngrok tcp 22
```

After running the command, you should see output similar to:

```
Session Status                online
Account                       your_email@example.com (Plan: Free)
Version                       3.20.0
Region                        India (in)
Latency                       50ms
Web Interface                 http://127.0.0.1:4040
Forwarding                    tcp://0.tcp.in.ngrok.io:10763 -> localhost:22
```

The **Forwarding** line is your public endpoint. In this example, it is:
- **Address:** `0.tcp.in.ngrok.io`
- **Port:** `10763`

---

## Step 4: Query the Public IP Address from the Terminal

To quickly fetch the current ngrok forwarding address (the TCP endpoint), run:

```bash
curl --silent http://localhost:4040/api/tunnels | grep -Eo 'tcp://[^"]+'
```

This command queries ngrok’s local web API and extracts the public TCP URL. You can then pass this URL to your friend.

---

## (Optional) Step 6: Set Up ngrok to Run on Boot

If you want ngrok to start automatically when your board boots up, create a systemd service.

1. **Create a Systemd Service File:**

   Open your editor with sudo privileges:

   ```bash
   sudo nano /etc/systemd/system/ngrok.service
   ```

   Paste the following configuration (adjust the `User` field if needed):

   ```ini
   [Unit]
   Description=ngrok TCP Tunnel for SSH
   After=network.target

   [Service]
   ExecStart=/usr/local/bin/ngrok tcp 22
   Restart=always
   User=ubuntu
   Environment=NGROK_LOG=stdout

   [Install]
   WantedBy=multi-user.target
   ```

2. **Reload Systemd and Enable the Service:**

   ```bash
   sudo systemctl daemon-reload
   sudo systemctl enable ngrok
   sudo systemctl start ngrok
   ```

3. **Verify the Service is Running:**

   ```bash
   sudo systemctl status ngrok
   ```

   Then, check the public endpoint using:

   ```bash
   curl --silent http://localhost:4040/api/tunnels | grep -Eo 'tcp://[^"]+'
   ```

---


## Connecting via SSH

Others can connect to your board by running the following command:

```bash
ssh ubuntu@ip_address -p numbers
Eg:
ssh <username>@0.tcp.in.ngrok.io -p 10763
```

> **Important:** Each time ngrok restarts (or your board reboots), the public address (and port) may change. You’ll need to check and share the new address as described above.

---


## Final Notes

- **Ephemeral Address:**  
  Remember, on the free plan, the TCP address is not fixed and will change after a restart. You'll need to check the endpoint and relay it to your friend each time.

- **Security:**  
  Exposing SSH publicly comes with risks. Consider using SSH keys and disabling password authentication to secure your board.

- **JTAG Connection:**  
  Since you'll be accessing the board via a JTAG connection and then providing the IP address to your friend, ensure your communication method is secure.

---
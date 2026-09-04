# N8N Open Collective Webhook to WhatsApp (via WAHA)

[![GitHub Sponsors](https://img.shields.io/badge/GitHub%20Sponsors-Support-ea4aaa?style=for-the-badge&logo=github-sponsors)](https://github.com/sponsors/Deeplab-Studio)
[![Open Collective](https://img.shields.io/badge/Open%20Collective-Support-7FADF2?style=for-the-badge&logo=opencollective)](https://opencollective.com/deeplab-studio)

This project allows you to automatically catch donations coming from **Open Collective** and announce them in your WhatsApp community or chat groups, thanks to the integration of **n8n** running on **Docker** and **WAHA (WhatsApp HTTP API)**.

This way, you can instantly thank your donors and keep your community informed about donations automatically.

---

> [!IMPORTANT]  
> **Recommendation for SSL and Webhook Access**  
> For n8n to securely receive webhooks from the outside world (via Open Collective), it needs to be connected to a domain and have an **SSL certificate**. Therefore, it is **highly recommended** to set up a **Cloudflare Tunnel (Cloudflared)** and connect your system to a domain.

---

## 📋 Requirements
- Docker & Docker Compose
- [n8n](https://n8n.io/)
- [WAHA (WhatsApp HTTP API)](https://waha.devlike.pro/)
- Open Collective Account
- WhatsApp Account (The number to be used as a bot)
- *Recommendation:* Cloudflare Tunnel (For external access and SSL)

---

## 🐳 Quick Setup (Docker)

You can use the `docker-compose.yml` file below to get the project up and running as quickly as possible.

### 1. Create and copy your docker-compose.yml file
Copy the code below, create a `docker-compose.yml` file on your server, and paste it inside. Do not forget to adjust fields like `WEBHOOK_URL` and passwords according to your environment.

```yaml
version: '3.8'

services:
  n8n:
    image: n8nio/n8n
    container_name: n8n
    ports:
      - "56788:5678"
    environment:
      # --- AUTHENTICATION ---
      - N8N_BASIC_AUTH_ACTIVE=true
      - N8N_BASIC_AUTH_USER=admin
      - N8N_BASIC_AUTH_PASSWORD=password
      
      # --- EXTERNAL ACCESS (Set your domain here) ---
      - WEBHOOK_URL=https://n8n.yourdomain.com/ # e.g. https://n8n.yourdomain.com/
      
      # --- INTERNAL NETWORK SETTINGS ---
      - N8N_HOST=n8n
      - N8N_PORT=5678
      - N8N_PROTOCOL=http
      
      - NODE_ENV=production
      - TZ=Europe/Istanbul # Change to your local timezone (e.g. UTC, America/New_York)
    volumes:
      - ./n8n_data:/home/node/.n8n
    networks:
      - n8n-waha-net
    restart: unless-stopped

  waha:
    image: devlikeapro/waha
    container_name: waha
    ports:
      - "3000:3000"
      - "3001:3001"
    volumes:
      - ./tmp:/tmp
      - ./waha_data:/app/data
    environment:
      - WAHA_LICENSE_KEY=your-license-key-here
      - WHATSAPP_DEFAULT_ENGINE=NOWEB
      - WHATSAPP_HOOK_EVENTS=message
      
      # Dashboard Credentials
      - WAHA_DASHBOARD_ENABLED=true
      - WAHA_DASHBOARD_USERNAME=admin
      - WAHA_DASHBOARD_PASSWORD=password

      # API Key
      - WAHA_API_KEY=your-api-key-here # Set a secure API key here
      - WHATSAPP_SWAGGER_USERNAME=admin
      - WHATSAPP_SWAGGER_PASSWORD=password
      
      # Webhook URL pointing to your n8n workflow
      - WHATSAPP_HOOK_URL=https://n8n.yourdomain.com/webhook/your-webhook-id/waha
      - WAHA_SESSION_NAME=session1
      
      - TZ=Europe/Istanbul # Change to your local timezone
    networks:
      - n8n-waha-net
    restart: unless-stopped

networks:
  n8n-waha-net:
    driver: bridge
```

### 2. Start the containers by running the following command in the terminal:
```bash
docker-compose up -d
```

### 3. Verify that the system is running by accessing the n8n and WAHA interfaces
For n8n on the local network (e.g., `http://localhost:56788`) and WAHA (`http://localhost:3000`). Once your Cloudflare Tunnel settings are done, you can access it via `https://n8n.your-domain.com`.

---

## Importing the Workflow (Quick Start)

If you don't want to do a manual setup, you can quickly start by importing the ready-made template on n8n:

### 1. Log in to the n8n interface.
### 2. Create a new Workflow or open an existing one.
### 3. Click the "Import from File" option from the menu on the top right.
### 4. Select the OpenCollective.json file located in the repo.
### 5. Update the settings inside the nodes according to your own WAHA API and WhatsApp information.

---

## 🛠️ Manual Setup Steps

If you want to set it up step-by-step yourself and learn, you can follow the instructions below:

## n8n and WAHA Settings

### 1. Go to your installed n8n settings.
![Step 1](images/step_1.png)

### 2. Add the WAHA plugin in the Community Node settings.
![Step 2](images/step_2.png)

### 3. Type the required library (@devlikeapro/n8n-nodes-waha) and click Install.
![Step 3](images/step_3.png)

## Starting the Workflow

### 4. Click "Add first step" from the n8n homepage and type "Waha" in the search box.
![Step 4](images/step_4.png)

### 5. Select and add the Waha Trigger option.
![Step 5](images/step_5.png)

### 6. Copy the Test URL and Production URL addresses inside the Trigger.
![Step 6-1](images/step_6-1.png)
![Step 6-2](images/step_6-2.png)

## WAHA Dashboard Settings

### 7. Log in to the WAHA Dashboard (interface).
![Step 7-1](images/step_7-1.png)

### 8. Scroll down in the settings section, click Add Webhook, and paste the URLs you copied here.
![Step 7-2](images/step_7-2.png)

### 9. Go back to n8n and click Execute (or Listen for test event) to start listening.
![Step 6-3](images/step_6-3.png)

## WhatsApp Group and Message Test

### 10. Add the WhatsApp number you logged in as a bot (to WAHA) to the group and give it admin privileges.
Add it to the WhatsApp group or announcement channel where you want to share posts and make it an admin. While in listening mode, send a test message to the group to test the Webhook and copy the channel/group's **ID** at the same time.
![Step 16-4](images/step_16-4.png)

### 11. Observe that the Webhook Payload is caught by n8n in the incoming test message.
![Step 6-5](images/step_6-5.png)

## Message Read (Seen) Operation

### 12. Type Waha in the search box and add the "Send seen" node.
![Step 8](images/step_8.png)

### 13. Connect the two nodes (Trigger and Send seen) using the arrow.
![Step 9](images/step_9.png)
![Step 10](images/step_10.png)

## Open Collective Webhook Settings

### 14. Search for Webhook from the search box and add it.
![Step 11](images/step_11.png)

### 15. Copy the Test URL and Production URL in the Webhook and set the HTTP Method to "POST".
![Step 12-1](images/step_12-1.png)

### 16. Log in to your Open Collective account and go to the Settings -> Webhooks section.
![Step 12-2](images/step_12-2.png)

### 17. Click the "New Webhook" button.
![Step 12-3](images/step_12-3.png)

### 18. Enter the n8n Webhook URL you copied into the window that opens and save it.
![Step 12-4](images/step_12-4.png)

## Sending a Message (Send a text message)

### 19. Type "Waha" in n8n, select the "Send a text message" node and add it.
![Step 13](images/step_13.png)

### 20. Establish the arrow connection between the Webhook and "Send a text message".
![Step 14](images/step_14.png)

### 21. You can select and connect a "No Operation" node so the end of the flow is not left empty (Optional).
![Step 15](images/step_15.png)

### 22. Enter the "Send a text message" node and make the necessary settings.
![Step 16](images/step_16.png)

## System Test and Deployment

### 23. Wait for the data to arrive by clicking Execute on the Webhook node.
![Step 17-1](images/step_17-1.png)

### 24. Trigger the system by making a 1 dollar test donation.
The "Waha Trigger" data will already have arrived. You can do this for the Open Collective "Webhook" data.
![Step 17-2](images/step_17-2.png)

### 25. Fill in the Chat ID section.
Select "Waha Trigger" from the Session section, take the `payload -> from` value (group ID) from the incoming data and paste it as a string.
![Step 17-3](images/step_17-3.png)

### 26. Click the Text section and create the template for the message to be sent.
![Step 17-4](images/step_17-4.png)

### 27. Add the incoming data into your text using drag-and-drop.
(Donor name, amount, etc.) This way you can create dynamic messages.
![Step 18](images/step_18.png)

### 28. Congratulations!
Now donations can be viewed instantly in your WhatsApp community or group, and you can automatically send thank-you messages to donors.
![Step 19](images/step_19.png)
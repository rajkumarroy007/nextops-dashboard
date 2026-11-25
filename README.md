# NextOps Industrial - Dashboard UI
## Azure VM Deployment Guide (Nginx)

A step-by-step guide to build and host the **NextOps Dashboard** on an Azure Ubuntu VM using **Node.js**, **Nginx**, and static hosting under `/var/www/html`.

---

## 1. Open Port 80 in Azure NSG

To allow HTTP traffic to your dashboard:

1. Open the [Azure Portal](https://portal.azure.com).
2. Go to your Virtual Machine > **Networking** > **Network settings** (or your **Network Security Group**).
3. Under **Inbound port rules**, click **Add inbound rule**:
   - **Destination port ranges**: `80`
   - **Protocol**: `TCP`
   - **Action**: `Allow`
   - **Priority**: `1010`
   - **Name**: `Allow-HTTP-80`
4. Ensure **Port 22 (SSH)** is also allowed.

---

## 2. Connect to the Azure VM

From your local machine, SSH into your Azure VM using the Public IP:

```bash
ssh azureuser@<YOUR_AZURE_PUBLIC_IP>
```

---

## 3. Install Node.js & npm (v20 LTS)

Install Node.js 20.x LTS (recommended for Vite 8 and React 19) via the NodeSource repository:

```bash
# Update system packages
sudo apt update && sudo apt install -y curl git

# Add NodeSource Node.js 20 repository
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -

# Install Node.js and npm
sudo apt install -y nodejs
```

Verify installed versions:
```bash
node -v   # Output should be v20.x.x
npm -v    # Output should be v10.x.x
```

---

## 4. Clone the Repository

```bash
git clone <YOUR_REPO_URL>
cd nextops-dashboard
```

---

## 5. Update API Endpoint Details

The dashboard needs the public address of your **NextOps Query Service**.

### Option A: Using a `.env` file (Recommended)

Create a `.env` file in the project root:

```bash
nano .env
```

Add your backend API Public IP and port:

```env
VITE_API_BASE_URL=http://<YOUR_QUERY_SERVICE_PUBLIC_IP>:8000
VITE_COPILOT_API_BASE_URL=http://<YOUR_COPILOT_PUBLIC_IP>:8002
```

Save and exit (`Ctrl+O`, `Enter`, `Ctrl+X`).

### Option B: Directly edit `src/config/api.ts`

```bash
nano src/config/api.ts
```

Update line 6 with your Query Service endpoint:

```typescript
export const API_BASE_URL = import.meta.env.VITE_API_BASE_URL || 'http://<YOUR_QUERY_SERVICE_PUBLIC_IP>:8000';
```

---

## 6. Install Dependencies & Build

Install packages and create the production build:

```bash
# Install npm dependencies
npm install

# Build static production assets
npm run build
```

This compiles your application and outputs static assets into the `./dist` folder.

---

## 7. Install Nginx

Install and enable Nginx web server:

```bash
sudo apt install -y nginx
sudo systemctl enable nginx
sudo systemctl start nginx
```

---

## 8. Copy Build Artifacts to `/var/www/html`

1. Remove default Nginx placeholder files:
   ```bash
   sudo rm -rf /var/www/html/*
   ```

2. Copy the compiled assets from `dist/` to `/var/www/html`:
   ```bash
   sudo cp -r dist/* /var/www/html/
   ```

---

## 9. Test in Browser

Open your browser and navigate to:

```text
http://<YOUR_AZURE_PUBLIC_IP>
```

Your NextOps Industrial condition monitoring dashboard will load and connect to your live query service API.

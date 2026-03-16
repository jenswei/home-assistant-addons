# oPODSync Add-on for Home Assistant

**oPODSync** is a GPodder-compatible sync server for podcasts, packaged as a Home Assistant add-on.

## 📦 Features

- GPodder-compatible sync server
- builds directly from the official source archive at https://fossil.kd2.org/opodsync
- Easy integration with Home Assistant

## ⚙️ Installation

1. In Home Assistant, go to **Settings → Add-ons → Add-on Store**.
2. Click on the three dots in the upper right corner and select **Repositories**.
3. Add the following URL:
https://github.com/jenswei/home-assistant-addons


4. After adding the repository, the `oPODSync` add-on should appear in the list.
5. Click on it, then click **Install**.

## 🔧 Configuration

This add-on does not require any configuration by default. It runs the OPODSync server on port 8080.
The persistent config folder is exposed to home assistant. Also the opodsyc database is placed there.

## 🛠️ Usage

Once installed and started, OPODSync will be available on the configured port (default: 8080). You can connect podcast clients that support GPodder sync to this server.

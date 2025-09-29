# GPodder Add-on for Home Assistant

This add-on launches a GPodder.net server to synchronize podcasts across multiple devices.

## Installation

1. In Home Assistant, go to **Supervisor → Add-on Repositories**.
2. Add the repository using the GitHub URL where you upload this.
3. Install it like any other add-on.
4. Configure username, email, and password from the configuration tab.
5. Access the interface at `http://<your-ip>:80` or `:8081`.

## Configuration

```yaml
username: "jose"
password: "clave_secreta"
email: "jose@example.com"
```
The user is created automatically at startup.


Credits to user josejamilena where I found this Addon!

Native (recommended): run on the PBX server.

curl -fsSLO https://github.com/anvari-alphait/IssabelOpDesk/blob/main/install.sh
chmod +x install.sh

sudo ./install.sh                                                                  # LAN, self-signed certificate
sudo OPDESK_DOMAIN=opdesk.example.com OPDESK_LE_EMAIL=admin@example.com ./install.sh


The installer clones to /opt/OpDesk. It sets up Node.js 24, the database user and schema, the AMI user, nginx, the certificate and the opdesk systemd service, then builds and starts OpDesk. Add --yes for an unattended run.

Then:

Open https://<server> (add :8443 or whichever port you chose if it isn't 443).
Sign in as admin with the password you entered, or the default OpDesk@2026 if you pressed Enter. Change it from the profile menu.
Under Users, link users to their extensions so they get a softphone.
To update, re-run sudo ./install.sh from /opt/OpDesk. Your settings and port are kept. To uninstall, run sudo ./install.sh --uninstall (what it removes). Upgrading from the Python backend (1.x)? See the upgrade notes.

Docker:

git clone https://github.com/Ibrahimgamal99/OpDesk.git && cd OpDesk
cp .env.example .env && nano .env
docker compose up --build -d
The container uses host networking and serves HTTP on 127.0.0.1:8765. Put nginx in front of it for TLS, using nginx/opdesk.conf.example. The database user, schema and AMI user are created on the host: see Docker.

→ Full details, service commands and troubleshooting: Installation guide · Settings and .env: Configuration guide

Architecture

Component	Role
React frontend	The panel. Live state arrives over the /ws WebSocket; history and settings use the REST API. The softphone registers to Asterisk through /sip-ws.
NestJS backend	Node.js 24, TypeScript. Keeps one AMI connection, turns Asterisk events into presence, queue and call events, and sends each user the part their role may see. Serves the REST API and the built frontend, and pushes call data to your CRM. A small Python helper analyses recordings for talk and silence.
Asterisk	Still owns the dialplan. OpDesk uses AMI for monitoring and call control (originate, transfer, listen / whisper / barge).
nginx	Terminates TLS (the browser needs HTTPS for the microphone) and proxies to the backend and to Asterisk.
Database	Reads the PBX's asterisk config and asteriskcdrdb CDR (FreePBX and Issabel use the same names). Its own database holds users, groups, contacts, settings, API keys, the CRM delivery log and analytics rollups. Call history is read from the CDR, not copied.
Tech stack
Layer	Technologies
Backend	Node.js 24, NestJS 11 (Express 5), TypeScript, ws WebSockets, mysql2 (MySQL/MariaDB), bcryptjs + JWT, zod validation, exceljs (XLSX export), web-push; Python venv with onnxruntime + WebRTC VAD for talk/silence analysis
Frontend	React 18, TypeScript, Vite, SIP.js (WebRTC), Recharts, Framer Motion, Lucide icons, react-i18next
Telephony	Asterisk AMI, Asterisk HTTP/WebSocket (SIP over WSS), RFC 8599 push via FCM and APNs, VAPID Web Push
Deployment	nginx, systemd, Let's Encrypt (Debian, Rocky 8 / Issabel 5); or Docker Compose
Documentation
Guide	What it covers
Installation	Supported systems (Issabel 5, FreePBX 16/17), ports and Apache, native and Docker installs, running, updating, troubleshooting.
Configuration	backend/.env, the Settings page, roles and visibility, audit log, debugging.
Mobile softphone	QR sign-in, RFC 8599 wake-up, Firebase and APNs setup.
Analytics	Every KPI, how it is computed, and how to tune it.
API overview · Endpoints	Authentication, API keys and scopes, and every endpoint.
CRM push · Contact lookup	The outgoing call-data push and the incoming contact lookup.
The full index is in docs/. A running server also serves interactive API docs at /docs and /redoc, and the OpenAPI spec at /api/openapi.yaml.

Community & support
Questions and discussion: Telegram group · mailing list opdesk-dev@googlegroups.com
Bugs and feature requests: GitHub Issues. Include your PBX (Issabel / FreePBX and version) and the output of sudo journalctl -u opdesk -n 200.
Contributing: pull requests are welcome. Open an issue first for larger changes.
Author: Ibrahim Gamal · LinkedIn · ib.gamal.a@gmail.com
If OpDesk is useful to you, star the repo. OpDesk is licensed under the GNU GPL v3: you may use, modify and redistribute it, and anything you distribute that is based on it must be released under GPL v3 with its source. It is developed by Ibrahim Gamal, with AI-assisted tooling for boilerplate and acceleration.

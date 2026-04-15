# 🔥  Firewall pfSense —

> Lab de sécurité réseau : déploiement et configuration d'un firewall pfSense sous VirtualBox

---

## 📋 Objectifs

- Déployer un firewall pfSense sur VirtualBox
- Configurer des règles de filtrage réseau
- Contrôler différents types de trafic
- Superviser les logs
- Réaliser des tests d'intrusion contrôlés

---

## 🏗️ Architecture réseau

```
[ Kali Linux ]
  (Attaquant)
      |
   [NAT Network]
      |
  [ pfSense ]
  WAN ←→ LAN
      |
   [NAT Network]
      |
[ Ubuntu Server ]
   (Serveur)
```

> **Note :** Les deux interfaces pfSense utilisent un **NAT Network** (réseau NAT) dans VirtualBox. Kali et pfSense partagent le même NAT Network côté WAN, et pfSense et Ubuntu Server partagent le même NAT Network côté LAN.

<table>
  <tr>
    <td><img width="240" alt="image" src="https://github.com/user-attachments/assets/25114f85-17ea-4ed0-9cc9-c486c319f317" /></td>
    <td><img width="240" alt="image" src="https://github.com/user-attachments/assets/5493c578-cbc0-4128-95fc-4a5c5cb1186f" /></td>
    <td><img width="240" alt="image" src="https://github.com/user-attachments/assets/2e563578-ece6-4c40-bd87-d9247429efd1" /></td>
    <td><img width="240" alt="image" src="https://github.com/user-attachments/assets/25d0256f-1a0b-4d7d-b779-5a9f491b2c63" /></td>
  </tr>
</table>

## 🖥️ Machines virtuelles

| Machine | Rôle | Réseau |
|---|---|---|
| Kali Linux | Attaquant | NAT Network (WAN side) |
| pfSense | Firewall | Adapter 1 → NAT Network (WAN) / Adapter 2 → NAT Network (LAN) |
| Ubuntu Server | Serveur cible | NAT Network (LAN side) |

---

## ⚙️ Configuration VirtualBox

### pfSense
- **RAM :** 2 Go minimum
- **Adaptateur 1 (WAN) :** NAT Network — même réseau que Kali
- **Adaptateur 2 (LAN) :** NAT Network — même réseau qu'Ubuntu Server

### Kali Linux
- **Adaptateur :** NAT Network — même réseau que le WAN pfSense

### Ubuntu Server
- **Adaptateur :** NAT Network — même réseau que le LAN pfSense



---

## 📦 Installation pfSense

1. Télécharger l'ISO pfSense depuis [https://www.pfsense.org/download/](https://www.pfsense.org/download/)
2. Créer la VM dans VirtualBox (2 Go RAM, 2 interfaces réseau)
3. Lancer l'installation en mode standard

### Configuration initiale

| Interface | Config |
|---|---|
| WAN | DHCP (automatique) |
| LAN | `192.168.1.1/24` |

### Accès à l'interface Web

Depuis une machine sur le LAN, ouvrir :

```
https://192.168.1.1
```

| Champ | Valeur |
|---|---|
| Username | `admin` |
| Password | `pfsense` |


---

## 🛡️ Configuration des règles Firewall

### Politique de sécurité appliquée

- Tout bloquer par défaut
- Autoriser uniquement les services essentiels
- Bloquer explicitement FTP
- Autoriser le trafic contrôlé

### Règles LAN — `Firewall → Rules → LAN`

| Action | Protocole | Port | Description |
|---|---|---|---|
| ✅ Allow | TCP | 22 | SSH |
| ✅ Allow | TCP | 80 | HTTP |
| ✅ Allow | TCP | 443 | HTTPS |
| ✅ Allow | UDP | 53 | DNS |
| ✅ Allow | ICMP | any | Ping |
| ❌ Block | TCP | 21 | FTP |
| ✅ Allow | UDP | 5004 | RTP (VoIP) |

> ⚠️ **Important :** L'ordre des règles est crucial. Les règles **ALLOW doivent être placées avant les règles BLOCK**.

<table>
  <tr>
    <td><img width="480" alt="ddkdk" src="https://github.com/user-attachments/assets/04c40ce1-9fc2-4a17-8777-4aec5123c047" /></td>
    <td><img width="480" alt="skssiwi" src="https://github.com/user-attachments/assets/a30269e0-14d0-4ab3-a074-08b0a7fbbde0" /></td>
  </tr>
</table>

---

## 🔧 Installation des services sur Ubuntu Server

```bash
sudo apt update
sudo apt install apache2 vsftpd openssh-server
```

---

## 🧪 Tests à réaliser

### 1. Scan réseau (Nmap)

```bash
nmap -sS <IP_SERVEUR>
```

### 2. Test HTTP

```bash
curl http://<IP_SERVEUR>
```



### 3. Test SSH (doit fonctionner ✅)

```bash
ssh user@<IP_SERVEUR>
```

### 4. Test FTP (doit être bloqué ❌)

```bash
ftp <IP_SERVEUR>
```

### 5. Test DNS

```bash
dig google.com
```

### 6. Test RTP/UDP

```bash
nc -u <IP_SERVEUR> 5004
```



---

## 📊 Supervision des logs

Les logs sont accessibles depuis l'interface Web pfSense :

```
Status → System Logs → Firewall
```

Les connexions bloquées (notamment FTP port 21) doivent apparaître dans les logs.




---


# Configuration Tailscale avec IP Forwarding et NAT sur Proxmox (PVE)

Ce fichier documente la configuration appliquée sur le nœud Proxmox `pve` pour activer le routage via Tailscale et assurer la sécurité réseau.

---

## 1. Activation de l'IP Forwarding

Deux fichiers ont été configurés pour activer le forwarding IPv4 :

```bash
echo 'net.ipv4.ip_forward = 1' | tee -a /etc/sysctl.d/99-tailscale.conf
echo 'net.ipv4.ip_forward = 1' | tee -a /etc/sysctl.conf
sysctl -p /etc/sysctl.conf
sysctl -p /etc/sysctl.d/99-tailscale.conf
```

---

## 2. Configuration des règles iptables

### Autoriser le trafic Tailscale

```bash
iptables -I FORWARD -i tailscale0 -j ACCEPT
iptables -I FORWARD -o tailscale0 -j ACCEPT
```

### Activer le NAT vers le LAN (vmbr0)

```bash
iptables -t nat -I POSTROUTING -o vmbr0 -j MASQUERADE
```

---

## 3. Sauvegarde des règles

Les règles iptables sont sauvegardées dans `/etc/iptables/rules.v4` :

```bash
iptables-save > /etc/iptables/rules.v4
```

---

## 4. Vérification de l'état du réseau

Pour vérifier que tout est en place :

```bash
sysctl net.ipv4.ip_forward
iptables -L FORWARD -n -v
ss -tulwn | grep LISTEN
```

---

## 5. Sécurité

- Le **firewall PVE** est actif et bloque tout trafic non autorisé
- NAT uniquement sur le réseau Tailscale (`100.64.0.0/10`)
- Règles de DROP sur les paquets multicast/broadcast non nécessaires
- Port d’administration 8006 uniquement accessible depuis IP autorisées

---

## 6. Résultat attendu

- Accès aux machines du LAN via l’IP Tailscale (`100.x.x.x`)
- Routage correct via le nœud PVE
- Trafic non sollicité bloqué

# Configuration Réseau Proxmox + Tailscale

## Objectif

Configurer Proxmox (PVE) pour autoriser le forwarding IP et permettre à Tailscale de relayer le trafic.

## Étapes effectuées

1. **Activation de l’IP Forwarding**

```bash
echo 'net.ipv4.ip_forward = 1' | tee -a /etc/sysctl.d/99-tailscale.conf
echo 'net.ipv4.ip_forward = 1' | tee -a /etc/sysctl.conf
sysctl -p /etc/sysctl.conf
```

2. **Règles iptables**

```bash
iptables -I FORWARD -i tailscale0 -j ACCEPT
iptables -I FORWARD -o tailscale0 -j ACCEPT
iptables -t nat -I POSTROUTING -o vmbr0 -j MASQUERADE
iptables-save > /etc/iptables/rules.v4
```

3. **Vérifications**

- `sysctl net.ipv4.ip_forward` → doit afficher `= 1`
- `iptables -L FORWARD -n -v` → doit montrer les règles ACCEPT pour `tailscale0`

## Sécurité

- Ports d’administration limités par PVEFW
- Trafic non autorisé bloqué par les chaînes DROP
- MASQUERADE appliqué uniquement sur `vmbr0`

## Notes

- Cette configuration permet à PVE d’agir comme **exit node** ou **subnet router** pour Tailscale.
- À tester depuis un autre nœud Tailscale avec un `ping` vers une IP locale derrière PVE.

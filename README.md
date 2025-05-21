# reti-1
# Guida alla Configurazione di Nodi, Switch e Router in una Rete Linux

Questa guida descrive come configurare una semplice infrastruttura di rete composta da nodi, uno switch in modalità promiscua e un router con instradamento tra due reti. Tutti i comandi sono eseguibili in ambiente Linux.

---

## Topologia della Rete

```
[Nodo_A]---[Switch]---[Router1]---[Nodo_C]
     |         |            |
[Nodo_B]-------+------------+
```

- **Nodo_A**: indirizzo IP nella rete 10.0.0.0/24, con rotta verso 192.168.3.0/24 tramite Router1
- **Nodo_B**: indirizzo IP nella rete 10.0.0.0/24
- **Nodo_C**: indirizzo IP nella rete 192.168.3.0/24, con rotta verso 10.0.0.0/24 tramite Router1
- **Switch**: configurato in modalità promiscua per collegare tutti i dispositivi della rete 10.0.0.0/24 e il Router1
- **Router1**: gateway tra le due reti, con IP su entrambe le interfacce e forwarding IP abilitato

---

## Configurazione dei Nodi

### Nodo_A

```bash
# Collegamento fisico: cable-1
ip addr add 10.0.0.3/24 dev eth0
ip route add 192.168.3.0/24 via 10.0.0.1 dev eth0
```

### Nodo_B

```bash
# Collegamento fisico: cable-2
ip addr add 10.0.0.5/24 dev eth0
```

### Nodo_C

```bash
# Collegamento fisico: cable-5
ip addr add 192.168.3.5/24 dev eth0
ip route add 10.0.0.0/24 via 192.168.3.1 dev eth0
```

---

## Configurazione dello Switch (Linux Bridge in modalità promiscua)

Supponiamo di avere quattro interfacce di rete (`eth0`, `eth1`, `eth2`, `eth3`), ognuna collegata tramite i cavi alla topologia.

```bash
# Attiva la modalità promiscua sulle interfacce collegate
ip link set eth1 up
ip link set eth2 up
ip link set eth3 up

# Crea il bridge di livello 2
ip link add br0 type bridge

# Aggiungi le interfacce al bridge
ip link set eth0 master br0
ip link set eth1 master br0
ip link set eth2 master br0
ip link set eth3 master br0

# Attiva il bridge
ip link set br0 up
```

> **Nota:** La modalità promiscua è automaticamente attivata sulle interfacce quando vengono aggiunte a un bridge, ma se vuoi forzarla:
>
> ```bash
> ip link set eth0 promisc on
> ip link set eth1 promisc on
> ip link set eth2 promisc on
> ip link set eth3 promisc on
> ```

---

## Configurazione del Router1

Router1 collega le due reti e abilita l'instradamento IP.

```bash
# Collegamenti fisici: cable-4 (verso switch), cable-5 (verso Nodo_C)

# Configura IP sulle interfacce
ip addr add 10.0.0.1/24 dev eth0     # Verso lo switch
ip link set eth1 up
ip addr add 192.168.3.1/24 dev eth1  # Verso Nodo_C

# Abilita l'instradamento IP
echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf
sysctl -p
```

---

## Riassunto Rotte

- Nodo_A: per parlare con Nodo_C (192.168.3.0/24) passa per 10.0.0.1 (Router1)
- Nodo_C: per parlare con Nodo_A e Nodo_B (10.0.0.0/24) passa per 192.168.3.1 (Router1)
- Router1: ha IP su entrambe le reti e fa da gateway tra le due

---

## Note Finali

- Assicurati che le interfacce fisiche/cavi siano collegati correttamente.
- Se usi macchine virtuali o container, mappa le interfacce secondo la topologia.
- Puoi espandere questa guida per aggiungere firewall (`iptables`/`nftables`) o DHCP se necessario.

---

**Autore:**  
Niklaus1911

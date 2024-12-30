# Ovládání LED na základě připojení k portu 443 pomocí `nftables` na OpenWRT

Tento návod ukazuje, jak nakonfigurovat `nftables` na OpenWRT zařízení, aby se ovládala LED (`indicator-1`) na základě připojení k portu 443 (HTTPS). LED se rozsvítí při připojení a zhasne, když připojení skončí.

## Požadavky

- OpenWRT zařízení s přístupem k `nftables`.
- Funkční LED (`indicator-1`) připojená k zařízení.
- Přístup k souborovému systému zařízení (SSH nebo jiný způsob).

## Krok 1: Vytvoření pravidel pro `nftables`

1. **Vytvoření souboru s pravidly pro `nftables`:**

   Na OpenWRT vytvoříme nový soubor, který bude obsahovat pravidla pro monitorování připojení na port 443.

   Spusťte následující příkaz pro vytvoření souboru:

   ```bash
   sudo nano /etc/nftables.d/99-https-led.nft
   
2. **Přidejte následující pravidla do souboru:**

Tento soubor bude obsahovat pravidla pro sledování připojení na port 443 a logování těchto připojení:

```bash
table inet filter {
    chain input {
        type filter hook input priority 0; policy drop;

        # Povolit lokální připojení
        iif lo accept

        # Povolit již navázaná připojení
        ct state established,related accept

        # Monitorování nových připojení na port 443 (HTTPS) z vnější sítě
        tcp dport 443 ip saddr != 127.0.0.1 ct state new log prefix "HTTPS_CONNECTED: " accept

        # Pravidlo pro detekci ukončení připojení na port 443
        ct state established drop log prefix "HTTPS_DISCONNECTED: "
    }
}
```
Vysvětlení:
    `ct state new`: Detekuje nová připojení na port 443.
    `log prefix "HTTPS_CONNECTED: "`: Záznam připojení s prefixem "HTTPS_CONNECTED".
    `ct state established`: Sleduje existující připojení a ukončuje je.
    `log prefix "HTTPS_DISCONNECTED: "`: Loguje odpojení z portu 443.

3. **Načtěte konfiguraci `nftables`:**

Po uložení souboru načtěte pravidla do `nftables`:

```bash
sudo nft -f /etc/nftables.d/99-https-led.nft
```

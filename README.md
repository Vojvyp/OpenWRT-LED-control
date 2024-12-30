# Ovládání LED na základě připojení k portu 443 pomocí `nftables` na OpenWRT

Tento návod ukazuje, jak nakonfigurovat `nftables` na OpenWRT zařízení, aby se ovládala LED (`indicator-1`) na základě připojení k portu 443 (HTTPS). LED se rozsvítí při připojení a zhasne, když připojení skončí.

## Požadavky

- OpenWRT zařízení s přístupem k `nftables`.
- Funkční LED (`rgb:indicator-1`) připojená k zařízení.
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
*   `ct state new`: Detekuje nová připojení na port 443.
*   `log prefix "HTTPS_CONNECTED: "`: Záznam připojení s prefixem "HTTPS_CONNECTED".
*   `ct state established`: Sleduje existující připojení a ukončuje je.
*   `log prefix "HTTPS_DISCONNECTED: "`: Loguje odpojení z portu 443.

3. **Načtěte konfiguraci `nftables`:**

Po uložení souboru načtěte pravidla do `nftables`:

```bash
sudo nft -f /etc/nftables.d/99-https-led.nft
```
## Krok 2: Vytvoření skriptu pro ovládání LED 

1. **Vytvoření skriptu pro monitorování logů:**

Nyní vytvoříme skript, který bude monitorovat logy generované nftables a podle připojení a odpojení od portu 443 bude ovládat LED.

Spusťte následující příkaz pro vytvoření skriptu:

```bash
vi /etc/hotplug.d/iface/99-nft-led-monitor
```

2. **Přidejte následující kód pro skript:**

Tento skript bude sledovat logy a na základě připojení na port 443 bude zapínat a vypínat LED (rgb:indicator-1).

```bash
#!/bin/sh

# Cesta k LED indikátoru
LED_PATH="/sys/class/leds/rgb:indicator-1/brightness"

# Prefix pro připojení
LOG_PREFIX_CONNECTED="HTTPS_CONNECTED"
LOG_PREFIX_DISCONNECTED="HTTPS_DISCONNECTED"

# Funkce pro rozsvícení LED
turn_on_led() {
    echo 1 > $LED_PATH  # Zapne LED
}

# Funkce pro zhasnutí LED
turn_off_led() {
    echo 0 > $LED_PATH  # Vypne LED
}

# Monitorování logů pro hledání připojení
tail -F /var/log/messages | while read -r line; do
    if echo "$line" | grep -q "$LOG_PREFIX_CONNECTED"; then
        turn_on_led  # Zapne LED při připojení
    elif echo "$line" | grep -q "$LOG_PREFIX_DISCONNECTED"; then
        turn_off_led  # Vypne LED při odpojení
    fi
done
```
Vysvětlení:

*   Skript sleduje logy v `/var/log/messages` pro výskyty "HTTPS_CONNECTED" a "HTTPS_DISCONNECTED".
*   Na základě těchto logů se zapíná nebo vypíná LED indikátor.

3. **Udělejte skript spustitelný:**

Aby skript fungoval, je nutné mu udělit práva pro spuštění:

```bash
sudo chmod +x /etc/hotplug.d/iface/99-nft-led-monitor
```

## Krok 3: Načtení a restartování služeb

Po dokončení nastavení je nutné načíst pravidla `nftables` a restartovat službu pro aplikaci změn.

1. **Načtěte konfiguraci `nftables`:**

```bash
nft -f /etc/nftables.d/99-https-led.nft
```

2. **Monitorujte logy a ovládejte LED:**

Pokud chcete zkontrolovat, zda logy správně zaznamenávají připojení, použijte následující příkaz:

```bash
tail -f /var/log/messages
```

3. **(Pokud je potřeba) restartujte zařízení:**

Pokud chcete zařízení restartovat, použijte příkaz:

```bash
reboot
```

## Závěr

Tento návod vám ukázal, jak nastavit nftables pro sledování připojení na port 443 a ovládat LED (`rgb:indicator-1`) na zařízení s OpenWRT. LED se rozsvítí při připojení na port 443 a zhasne při jeho uzavření.


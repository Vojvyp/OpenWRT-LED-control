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

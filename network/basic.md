
# dns
## temporal modification

```bash
sudo resolvectl dns wlan0 1.1.1.1 1.0.0.1
```

## persistent change

```bash
sudo nmcli connection modify "Wi-Fi" ipv4.dns "1.1.1.1 1.0.0.1"
```

## flush caches
```bash
sudo resolvectl flush-caches
sudo resolvectl statistics
```

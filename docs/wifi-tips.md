# Running a server over Wi-Fi

Tunnel latency and throughput depend on your local link and ISP upload speed.

1. **Prefer Ethernet** for a machine that acts as a server. Wi-Fi is the most common source of jitter and drops.
2. If you must use Wi-Fi: use 5 GHz or 6 GHz, sit close to the router, and pick a non-congested channel.
3. Test **upload** speed (speedtest.net); that is what visitors download from you.
4. Disable PC sleep and Wi-Fi power saving, or the tunnel goes offline.
5. Close background sync/update jobs that compete for upload bandwidth.
6. Optional: use `1.1.1.1` as DNS; it speeds up lookups but not transfers.

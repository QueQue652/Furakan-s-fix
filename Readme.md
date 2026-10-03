(The terminal commands are for macos, if youre on windows it may vary)

1. put kick-multi7.py on your kick-viewer-torproxy-main folder then run it on the terminal, if it doesn't work follow the instructions below.

2. Open `Terminalfix.sh` and copy "the entire content" — from the first line `cat > kick-multi7.py << 'KICK_EOF'` down to the final `KICK_EOF` line.
3. Paste it into the terminal and press **Enter**. This rewrites `kick-multi7.py` byte-for-byte with correct syntax.
4. Verify the fix (no output = clean file):

   python3 -m py_compile kick-multi7.py

5. Run the script again:

   python3 kick-multi7.py


---

## ⏳ Why reaching 100+ connections might takes a few minutes

This is expected behavior, not a hang. The startup pipeline is deliberately staged:

1. **Container provisioning** – Docker builds the proxy image and launches each container with 6 SOCKS ports.
2. **Network bootstrap (60s wait)** – every proxy instance inside the containers must establish its own encrypted circuits before it can carry traffic.
3. **IP rotation loop** – edge protection services (Cloudflare) reject many proxy exit IPs on first contact. The script automatically retries and rotates through all available ports until it reaches the API from a clean IP. On a bad network wave this can take dozens of attempts.
4. **Token pre-fetching** – before any socket is opened, the script fills a pool of ~150+ authenticated handshake tokens, each fetched through a different proxy route.
5. **Batched socket opening** – connections are opened in small, rate-limited batches per port to keep sessions stable and avoid overload, so the counter climbs gradually rather than instantly.

Just leave the program running and watch the live stats panel (`Conn`, `Attempts`, `TokenPool`). Connections accumulate steadily once the pool is warm. 

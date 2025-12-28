# next.js

# Run on Port 80


0. Find the linked Node binary:

   ```bash
   which node
   ```


1. Find the real Node binary:

```bash
readlink -f /run/user/1000/fnm_multishells/2076_1764275467596/bin/node
```

You should get something like:

```text
/home/youruser/.fnm/node-versions/v22.0.0/installation/bin/node
```


2. Give that binary permission to bind to low ports:

```bash
sudo setcap 'cap_net_bind_service=+ep' /home/youruser/.fnm/node-versions/vXX.XX.X/installation/bin/node
```

(replace the path with what you got from `readlink -f`)


3. Verify:

```bash
getcap /home/youruser/.fnm/node-versions/vXX.XX.X/installation/bin/node
# should output: cap_net_bind_service=ep
```


4. Now run Next.js as normal user:

```bash
PORT=80 npm run dev
# or

npx next dev -p 80
```

**Notes:**

* If you install a new Node version via `fnm`, repeat the `setcap` for that version's `node` binary.
* You must run `setcap` on the actual binary, not the `/run/user/...` symlink.


---
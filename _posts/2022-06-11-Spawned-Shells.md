---
title: Spawned Shells
published: true
---

# SPAWNED SHELLS

```bash
python -c 'import pty; pty.spawn("/bin/sh")'

python3 -c 'import pty; pty.spawn("/bin/sh")'
```

```bash
echo os.system('/bin/bash')
```

```bash
/bin/sh -i
```

```bash
perl —e 'exec "/bin/sh";'

perl: exec "/bin/sh";
```

```bash
ruby: exec "/bin/sh"
```

```bash
lua: os.execute('/bin/sh')
```
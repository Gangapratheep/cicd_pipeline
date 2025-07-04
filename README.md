The error you're seeing:

```
Error response from daemon: bridge is a pre-defined network and cannot be removed
```

means that `bridge` is one of Docker’s **default networks**, and **Docker does not allow removing default networks** like:

* `bridge`
* `host`
* `none`

These are **built-in** and are essential for Docker’s operation.

---

### 🔍 Default Docker Networks:

| Name     | Type   | Description                                     |
| -------- | ------ | ----------------------------------------------- |
| `bridge` | bridge | Default network for containers on a single host |
| `host`   | host   | Shares the host’s networking stack              |
| `none`   | null   | Disables networking for a container             |

You **can only remove user-defined networks**, like this one from your earlier output:

```
578e6511f261   node_project_node_mysql_net   bridge    local
```

---

### ✅ So in summary:

* You **cannot delete** the `bridge`, `host`, or `none` networks.
* You **can delete** custom networks (those you create with `docker network create`) **if they’re not in use**.

Let me know if you want to list containers using a network or clean up all unused networks.

## Architecture
- Kubernetes follows a control plane-worker node architecture
- Control Plane (the brain): makes global decisions and manages the cluster's state.
- Worker Nodes (The Muscle): run the actual application workloads.
## The SSH "Node-Hopping" Rule
- assume you must move: most tasks require switching from the base node to a worker or control plane node.
- Verify Your Prompt: Always double-check user@node before executing commands.
- THe Power Move: Use sudo -i immediately after SSH-ing to fain root access.
- Essential for: Editing kube-apiserver.yaml and other system files.
## Optimizing the Browser
- The Two-Tab Rule:
  - Exam Environment
  - Official Documentation (kubernetes.io/docs)
- Search Strateties: Use the internal site search for rapid access to Gateway API or NetworkPolicy templates.
- Manual Navitgation: Don't rely on personal bookmarks;
  they won't be there. Know the doc structure by heart.

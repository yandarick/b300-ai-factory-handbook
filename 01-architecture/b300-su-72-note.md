# B300 Scalable Unit sizing

Current NVIDIA DGX B300 SuperPOD XDR guidance uses 72 DGX B300 systems per scalable unit (SU). One SU uses 8 compute leaf switches and 4 compute spine switches. Two SUs provide 144-node capacity with 16 leaf and 8 spine switches. For a 128-node project, plan the fabric for 144 nodes and leave 16 node positions unused for future expansion. Last verified: 2026-10-03.

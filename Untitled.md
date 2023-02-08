

Usually, a process will be cooperative if it needs to share information, speed up computation, allow modularity, or by conveneience. To accomplish this, a cooperating process needs to affect or be affected by other processes, including sharing data. That is, the cooperating process need **interprocess communication**. Two models of the IPC is **sharing memory** and **passing messages**.

One way to think about this is through the *producer-consumer* problem (AKA **bounded-buffer problem**). A *producer* process produces information that is consumed by a *consumer* process.
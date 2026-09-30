# E.C.H.O. – Seismic Monitoring

Distributed, fault-tolerant platform for real-time classification of seismic events (FFT, replicated processing, idempotent persistence).

Group project (5 people) for Advanced Programming, MSc in Engineering in Computer Science and AI, Sapienza University of Rome (2025/26).

> Portfolio copy of the team's official repository [enoughpaladin00/2003424_ECHO](https://github.com/enoughpaladin00/2003424_ECHO), with full commit history.

## My role
I built the ingestion broker (`source/broker`): sensor discovery, one ingestion loop per sensor, reconnection with exponential backoff, dead-letter queue for invalid messages, fan-out to all active processing replicas.

Team: Fabiano Cacioli · Jacopo Rossi · Fabrizio Pietrobono · Emanuele Smisi · Luca Buonomini

mermaid
'''graph TD
    subgraph Containerfile
        A[Start: FROM Base Image] --> B(Layer 1: Minimal OS/Kernel);
        B --> C(Layer 2: RUN Install Dependencies);
        C --> D(Layer 3: COPY Application Code);
        D --> E(Layer 4: Configuration / Entrypoint);
    end
    E -- Result --> F{Final Container Image};
    style F fill:#aaffdd,stroke:#333,stroke-width:2px
    click A "Explain FROM/Base"
    click C "Explain RUN/Dependencies"
    click D "Explain COPY/Code"
    '''

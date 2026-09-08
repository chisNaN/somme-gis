# Architecture — somme-gis

## Pipeline de données

```mermaid
flowchart TD
    A["BD TOPO® (IGN)<br/>GPKG 7z, cartes.gouv.fr"] -->|ogr2ogr import| B["PostGIS (postgis_container)<br/>commune, cours_d_eau, commune_label_point"]
    B --> C["Martin<br/>Diffusion MVT via HTTP :3000"]

    style A fill:#D3D1C7,stroke:#5F5E5A,color:#2C2C2A
    style B fill:#9FE1CB,stroke:#0F6E56,color:#04342C
    style C fill:#9FE1CB,stroke:#0F6E56,color:#04342C
```

## Points d'entrée front

`index.html` charge un style unique généré par Maputnik. `index.htm` assemble ses couches à la main en JavaScript directement depuis Martin, sans passer par Maputnik.

```mermaid
flowchart TD
    M["Martin<br/>Tuiles vectorielles :3000"]
    MP["Maputnik<br/>Éditeur de style :8888"]
    SJ["styles/somme.json<br/>Style exporté par Maputnik"]
    IH["index.html ★<br/>MapLibre GL JS — style unique"]
    SB["Style de base public<br/>ex: demotiles, openfreemap"]
    IM["index.htm<br/>Couches ajoutées à la main"]

    M -->|aperçu des tuiles| MP
    MP --> SJ
    SJ --> IH
    M -->|sources via addSource| IM
    SB --> IM

    linkStyle 0 stroke:#7F77DD,stroke-width:2px
    linkStyle 1 stroke:#7F77DD,stroke-width:2px
    linkStyle 2 stroke:#7F77DD,stroke-width:2px

    style IH fill:#7F77DD,stroke:#3C3489,color:#fff
    style MP fill:#F5C4B3,stroke:#993C1D,color:#4A1B0C
    style SJ fill:#F5C4B3,stroke:#993C1D,color:#4A1B0C
    style M fill:#9FE1CB,stroke:#0F6E56,color:#04342C
    style SB fill:#D3D1C7,stroke:#5F5E5A,color:#2C2C2A
    style IM fill:#D3D1C7,stroke:#5F5E5A,color:#2C2C2A
```

**Chaîne violette** = point d'entrée privilégié (`index.html`), qui passe par le style JSON exporté depuis Maputnik.

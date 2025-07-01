# Trivy CI/CD Pipeline med VEX og .trivyignore

Dette repository er et Proof-of-Concept (PoC), der demonstrerer, hvordan man opsætter en automatiseret DevSecOps-pipeline ved hjælp af GitHub Actions og Trivy. Pipelinen scanner et container-image for sårbarheder og anvender både VEX (Vulnerability Exploitability eXchange) og `.trivyignore` til at håndtere og undertrykke fund.

Det primære image, der scannes, er `docker.redpanda.com/redpandadata/console:v2.8.5`, som i forvejen har specifikke CVE'er, som jeg kender til. 

## Formål

Projektet har til formål at demonstrere:
1.  Opsætning af en grundlæggende GitHub Actions-pipeline, der scanner et image på `push`.
2.  Korrekt installation og brug af Trivy CLI i et CI/CD-miljø.
3.  Anvendelse af et `openvex.json`-dokument til formelt at erklære status på specifikke sårbarheder.
4.  Anvendelse af en `.trivyignore`-fil som en pragmatisk løsning til at undertrykke sårbarheder.

## Konfiguration og filer

* **`.github/workflows/security-scan.yml`**: Indeholder definitionen for den automatiserede pipeline. Pipelinen kører på `ubuntu-latest` og følger en pålidelig to-trins-raket:
    1.  **Install Trivy:** Installerer en specifik version af Trivy ved hjælp af det officielle `install.sh`-script.
    2.  **Run Trivy Scan:** Kalder `trivy`-kommandoen direkte for at sikre fuld kontrol over parametre.

* **`openvex.json`**: Anvendes til formel dokumentation af sårbarheds-status. I dette projekt undertrykker den:
    * `CVE-2025-31498` i `c-ares`-biblioteket med status `not_affected`.
    * `CVE-2025-22872` i `golang.org/x/net`-pakken med status `not_afffected`

* **`.trivyignore`**: Anvendes som en simpel og direkte måde at undertrykke sårbarheder på. I dette projekt undertrykker den:
    * `CVE-2025-0913` i `stdlib`-pakken (se "Vigtige Lærdomme" nedenfor).

## Vigtige Lærdomme og Workarounds

Under opsætningen af dette projekt stødte jeg på en specifik og genstridig udfordring med Trivy (testet op til v0.52.2).

**Problem:** Det var ikke muligt at undertrykke en sårbarhed (`CVE-2025-0913`) i Go's standardbibliotek (`stdlib`) ved hjælp af en VEX-fil.

**Fejlfinding viste:**
* Syntaksen i `openvex.json` var korrekt.
* PURL-identifikatoren (`pkg:golang/stdlib@v1.23.7`) var 100% identisk med den, som Trivy selv rapporterede.
* Andre VEX-regler for ikke-`stdlib`-pakker i samme fil virkede som forventet.

**Konklusion og Workaround:**
Den eneste plausible konklusion er, at der findes en bug eller et uofficielt "quirk" i, hvordan Trivy matcher VEX-regler mod `stdlib`.

Den valgte løsning er en pragmatisk workaround:
> Sårbarheden `CVE-2025-0913` blev flyttet fra `openvex.json` til `.trivyignore`-filen, som pålideligt undertrykker den. Hertil vil man kunne indkludere en `exp:` linje for at tilføje denne ignorering efter noget tid, hvis nu fejlen skulle være blevet løst. 


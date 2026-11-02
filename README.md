# Vertex Corp Lab — cible d'entraînement au test d'intrusion

> ⚠️ **AVERTISSEMENT — À LIRE AVANT TOUT**
>
> Cette application est **délibérément vulnérable**. Elle sert **uniquement** de
> terrain d'entraînement pédagogique, **sur votre propre machine**, dans le cadre
> d'un **contrat signé** de non-pratique illégale.
>
> - Elle se lance **en local uniquement** : tous les ports sont liés à `127.0.0.1`.
> - **Ne JAMAIS** l'exposer sur Internet ou un réseau partagé, ni changer les liaisons
>   de ports en `0.0.0.0`.
> - On n'attaque **que cette application**, **sur sa propre machine**.
> - Objectif : **comprendre les failles pour mieux défendre** (chaque faille est
>   accompagnée de sa contre-mesure dans `guide/`).

Vertex Corp Lab est une application web **Laravel 11** volontairement faillible,
packagée en **Docker**, dans l'esprit de **DVWA** / **OWASP Juice Shop**. Fil narratif :
*« infiltrez l'application interne de Vertex Corp »*. Le parcours suit les **5 phases
d'un pentest** et propose **15 flags** (3 par phase : avant → pendant → après),
au format `FTM{...}`.

---

## 1. Prérequis : installer Docker

- **Windows / macOS** : installez **Docker Desktop** → <https://www.docker.com/products/docker-desktop/>
- **Linux** : installez **Docker Engine** + plugin Compose → <https://docs.docker.com/engine/install/>

Vérifiez l'installation :

```bash
docker --version
docker compose version
```

> Sur Windows/macOS, **Docker Desktop doit être démarré** (icône baleine active) avant de continuer.

## 2. Récupérer le lab

```bash
git clone <URL_DU_DEPOT>
cd vertex-corp-lab
```

## 3. Préparer la configuration

```bash
cp .env.example .env
```

(Le lab fonctionne avec les valeurs par défaut ; vous pouvez laisser le `.env` tel quel.)

## 4. Démarrer (une seule commande)

```bash
docker compose up -d --build
```

Au premier lancement, l'image se construit, la base MySQL démarre, puis les
**migrations + le seed** s'exécutent automatiquement (patientez ~30–60 s).
Suivez l'avancement avec :

```bash
docker compose logs -f app
```

Attendez la ligne : `[vertex] Pret. Ouvrez http://localhost:8000`.

## 5. Ouvrir l'application

👉 <http://localhost:8000>

## 6. Attaquer depuis Kali & soumettre les flags

Depuis votre machine Kali (même machine, ou même réseau local **privé**), les outils
s'appliquent à la surface **locale** du lab. Exemples de point de départ :

```bash
nmap -sV 127.0.0.1
whatweb http://localhost:8000
gobuster dir -u http://localhost:8000 -w /usr/share/wordlists/dirb/common.txt
nikto -h http://localhost:8000
```

Le détail phase par phase (outils, commandes, flags, défense) est dans **`guide/`**
(`vertex-corp-lab-guide.md` et le PDF). Soumettez vos 15 flags sur **FTM Range**.

## 7. Arrêter / réinitialiser

```bash
# Arrêter (conserve la base)
docker compose down

# Repartir sur une base 100 % propre (supprime les volumes : DB + code)
docker compose down -v && docker compose up -d --build
```

> 💡 Si vous **modifiez le code** du lab, faites un `docker compose down -v` avant de
> relancer : le code est stocké dans un volume Docker qui n'est rafraîchi que lorsqu'il
> est vide.

---

## Dépannage

| Symptôme | Cause probable | Solution |
|---|---|---|
| `Ports are not available` / `bind: address already in use` | Le port **8000** ou **3306** est déjà pris | Arrêtez l'autre service, **ou** changez le port hôte dans `docker-compose.yml` (ex. `127.0.0.1:8080:80`) puis relancez. |
| `Cannot connect to the Docker daemon` | Docker **n'est pas démarré** | Lancez Docker Desktop (Win/Mac) ou `sudo systemctl start docker` (Linux). |
| La page affiche **502 Bad Gateway** au tout début | L'app finit encore de migrer/seed | Attendez quelques secondes et rechargez ; vérifiez `docker compose logs -f app`. |
| Erreurs de **permissions** sur `storage/` | Droits du volume | `docker compose down -v && docker compose up -d --build`. |
| Je veux **réinitialiser la base** | Données modifiées pendant les tests | `docker compose down -v && docker compose up -d --build`. |
| `docker compose` inconnu (ancienne version) | Ancien Docker | Mettez à jour, ou utilisez `docker-compose` (tiret). |

---

## Structure du dépôt

```
vertex-corp-lab/
├── docker-compose.yml        # app + web (nginx) + db — ports liés à 127.0.0.1
├── docker/                   # Dockerfile, nginx.conf, entrypoint.sh
├── app-overlay/              # code Laravel volontairement vulnérable (// VULN: ...)
│   ├── routes/ app/ database/ resources/ public/ bootstrap/
├── guide/                    # guide pédagogique (Markdown + PDF) : 5 phases, Défi/Solution, Côté défense
├── .env.example
├── .gitignore                # ignore .env ET FLAGS.md
└── README.md
```

> **Note formateur** : le corrigé `FLAGS.md` (les 15 flags + obtention) est **ignoré par git**
> et n'est donc **pas** publié sur GitHub. Il est distribué séparément aux formateurs.

---

## Rappel final

Ce lab est un **outil pédagogique**. Les techniques apprises ici ne doivent être
employées que sur des systèmes que vous **possédez** ou pour lesquels vous disposez
d'une **autorisation écrite**. **Ne jamais exposer cette application sur Internet.**

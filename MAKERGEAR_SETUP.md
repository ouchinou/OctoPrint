# MakerGear Fork Setup Guide

## État Actuel

✅ **Fork créé avec succès!**
- **Base**: OctoPrint 1.11.4 (main branch)
- **Branche**: `makergear/stable`
- **Remote origin**: `git@github.com:ouchinou/OctoPrint.git`
- **Remote upstream**: `https://github.com/OctoPrint/OctoPrint.git`

## Prochaines Étapes

### 1. Appliquer les Personnalisations MakerGear

#### 📢 Announcement Feed
Modifier le flux d'annonces pour pointer vers MakerGear:

```bash
# Fichier: src/octoprint/plugins/announcements/__init__.py
```

**Changement requis**:
```python
# Ligne ~60
settings = dict(channels=dict(
    _important=dict(
        name="MakerGear Announcements",  # Au lieu de "Important Announcements"
        description="MakerGear announcements about OctoPrint.",
        priority=1,
        type="rss",
        url="http://setup.makergear.com/feeds/important.xml"  # Au lieu de octoprint.org
    ),
    # ... reste des channels
))
```

#### ⚙️ Settings Par Défaut
Créer un fichier de configuration par défaut MakerGear:

```bash
# Créer: src/octoprint/plugins/corewizard/makergear_defaults.yaml
```

**Contenu**:
```yaml
serial:
  timeout:
    detection: 0.5  # Plus rapide pour MakerGear
    temperatureAutoreport: 0  # Désactivé pour firmware MakerGear
  supportResendsWithoutOk: false  # Firmware MakerGear
  autoUppercaseBlacklist:
    - M117  # Préserver casse pour LCD

server:
  host: "0.0.0.0"  # Accessible réseau par défaut (optionnel)
```

#### 🧙 Wizard Simplifié (Optionnel)
Si vous voulez simplifier le wizard, modifier:

```bash
# Fichier: src/octoprint/plugins/corewizard/__init__.py
```

**Ajouter** avant `def _get_subwizard_attrs`:
```python
from octoprint.settings import settings as s

def _ensure_defaults(self):
    """Auto-configure settings pour MakerGear"""
    if s().get(["server", "pluginBlacklist", "enabled"]) is None:
        s().set(["server", "pluginBlacklist", "enabled"], True)
        s().save()
    
    if s().get(["server", "onlineCheck", "enabled"]) is None:
        s().set(["server", "onlineCheck", "enabled"], True)
        s().save()
```

### 2. Créer une Branche de Travail

```bash
# Créer branche pour personnalisations
git checkout -b feature/makergear-customizations

# Appliquer les modifications ci-dessus
# Puis commiter
git add .
git commit -m "feat(makergear): Add MakerGear customizations

- Announcement feed pointing to setup.makergear.com
- Temperature autoreport disabled by default
- Optimized serial timeouts
- Simplified wizard for MakerGear users"
```

### 3. Tester les Modifications

```bash
# Installer en mode développement
pip install -e .

# Lancer OctoPrint
octoprint serve --debug
```

**Tests à effectuer**:
- [ ] Vérifier announcement feed MakerGear
- [ ] Tester connexion série avec imprimante MakerGear
- [ ] Vérifier température (pas d'autoreport)
- [ ] Tester wizard d'installation
- [ ] Vérifier plugins MakerGear (Setup, Mglcd)

### 4. Merger dans Stable

```bash
# Si tests OK, merger dans stable
git checkout makergear/stable
git merge feature/makergear-customizations

# Tag version
git tag v1.11.4-mg1
```

### 5. Pusher vers Votre Fork

```bash
# Pusher la branche stable
git push origin makergear/stable

# Pusher le tag
git push origin v1.11.4-mg1
```

---

## Structure des Branches

### Branche Production: `makergear/stable`
- **Base**: `upstream/main` (stable releases)
- **Usage**: Déploiement production utilisateurs
- **Merges**: Lors des releases upstream (1.11.4 → 1.11.5)

### Branche Dev: `makergear/next` (Optionnel)
```bash
# Si vous voulez tester features en avance
git checkout -b makergear/next origin/dev
```
- **Base**: `upstream/dev` (développement)
- **Usage**: Tests internes nouvelles features
- **Merges**: Hebdomadaire ou bi-hebdomadaire

---

## Maintenance Continue

### Synchroniser avec Upstream

```bash
# Tous les 1-2 mois (lors des releases OctoPrint)

# 1. Fetch upstream
git fetch upstream

# 2. Vérifier nouvelle version
git log makergear/stable..upstream/main --oneline

# 3. Créer branche de merge
git checkout -b merge/upstream-1.11.5 makergear/stable

# 4. Merger
git merge upstream/main

# 5. Résoudre conflits si nécessaire
# 6. Tester
# 7. Merger dans stable
git checkout makergear/stable
git merge merge/upstream-1.11.5

# 8. Tag
git tag v1.11.5-mg1
git push origin makergear/stable v1.11.5-mg1
```

---

## Personnalisations Recommandées

### ✅ Critiques (À Appliquer)
1. **Announcement feed** → `setup.makergear.com`
2. **Temperature autoreport** → Désactivé (0)
3. **Detection timeout** → 0.5s
4. **ResendWithoutOk** → False

### ⚠️ Optionnelles
1. **Wizard simplifié** → Auto-config checks
2. **Host 0.0.0.0** → Accès réseau par défaut
3. **UI/Branding** → Logos MakerGear

### ❌ À Éviter
1. Retirer paramètres série upstream
2. Retirer plugins modernes (appkeys, backup, etc.)
3. Hardcoder des valeurs (utiliser config.yaml)

---

## Références

📄 **ANALYSE_FORK_MAKERGEAR.md** - Historique complet du fork  
📄 **CORRECTIONS_FORK_VS_UPSTREAM.md** - Analyse corrections  
📄 **PERSONNALISATIONS_MAKERGEAR.md** - Détails personnalisations  
📄 **GUIDE_FORK_MAIN_VS_DEV.md** - Stratégie main vs dev

---

## Aide Rapide

### Vérifier état actuel
```bash
git status
git log --oneline -5
```

### Voir différence avec upstream
```bash
git log makergear/stable..upstream/main --oneline
```

### Annuler modifications locales
```bash
git reset --hard HEAD
```

### Créer nouvelle branche feature
```bash
git checkout -b feature/nom-feature makergear/stable
```

---

*Guide créé le: 14 novembre 2025*  
*Base: OctoPrint 1.11.4 (main)*  
*Fork: github.com/ouchinou/OctoPrint*

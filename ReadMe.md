# LAB 3 — Observation du trafic HTTP(S) Android avec Burp Suite

**Cours : Sécurité des applications mobiles**

---

## Vue d'ensemble

Ce laboratoire met en place un proxy d'observation entre un Android Emulator et une application cible autorisée (site de test ou application de formation). Il vise à comprendre comment un proxy s'insère dans le chemin réseau, ce que Burp Suite capture (requêtes, en-têtes, cookies, paramètres), et comment documenter rigoureusement les observations dans un contexte d'audit mobile.

Le lab repose sur une approche passive et pédagogique : l'accent est mis sur la lecture et la compréhension du trafic, non sur sa manipulation.

---

## Objectifs pédagogiques

À l'issue de ce laboratoire, l'étudiant est en mesure de :

1. Vérifier qu'un navigateur Android achemine son trafic via Burp Suite.
2. Identifier les éléments constitutifs d'une requête HTTP : URL, méthode, en-têtes, cookies, paramètres.
3. Distinguer HTTP et HTTPS, et expliquer le rôle d'un certificat CA dans un environnement de laboratoire.
4. Produire une trace d'audit structurée comprenant des preuves et leur contexte.

---

## Prérequis

- Burp Suite Community installé sur la machine hôte.
- Un Android Emulator fonctionnel (par exemple, un appareil virtuel de type Nexus ou Pixel configuré dans Android Studio).
- Un environnement réseau de laboratoire isolé et une cible autorisée (site de test interne, application de formation, ou maquette locale).

---

## Règles de sécurité (obligatoires)

Ces règles s'appliquent à l'intégralité de la séance et ne souffrent d'aucune exception :

- Le trafic intercepté doit être exclusivement issu de cibles préalablement autorisées dans le cadre du lab.
- Aucun compte personnel, aucune session réelle, et aucune donnée sensible ne doivent transiter par le proxy.
- Tout certificat de laboratoire installé sur l'émulateur doit être retiré en fin de séance, conformément à la procédure de nettoyage décrite en dernière section.

---

## Étapes du laboratoire

### Étape 1 — Préparer Burp Suite (projet et mode Proxy)

1. Lancer Burp Suite.
2. Créer ou ouvrir un projet temporaire dédié au laboratoire.
3. Naviguer vers l'onglet Proxy.
4. Vérifier que la page Intercept est visible et que l'état affiché est "Intercept is off" — cet état est volontaire au démarrage.

Pourquoi ? Un proxy d'observation ne doit pas interrompre le flux réseau tant que la configuration n'a pas été validée. Démarrer avec l'interception activée introduit des blocages non productifs et rend l'environnement instable.

À observer :
- La présence de l'onglet HTTP history, qui liste les requêtes capturées.
- L'accessibilité du bouton d'activation/désactivation de l'interception.

Erreurs fréquentes :
- Démarrer avec l'interception activée, bloquant ainsi toute navigation dans l'émulateur.
- Se perdre entre les onglets Proxy, Target et Repeater sans avoir préalablement vérifié que la capture fonctionne.

---

### Étape 2 — Vérifier le Proxy Listener (adresse et port)

1. Dans Burp Suite, ouvrir Proxy settings ou Proxy listeners selon la version installée.
2. Vérifier qu'un listener est actif (statut Enabled).
3. Noter les paramètres suivants :
   - Port du proxy : `<PORT_PROXY>`
   - Adresse d'écoute : Loopback only ou All interfaces, selon la configuration réseau du lab.

Pourquoi ? L'émulateur Android doit connaître avec précision l'adresse et le port vers lesquels rediriger son trafic. Un listener inactif ou mal configuré rend la capture impossible.

À observer :
- Un listener dans l'état running.
- Un port clairement défini et documenté (ex. : 8080 — à consigner sous la forme `<PORT_PROXY>` dans les livrables).

Erreurs fréquentes :
- Listener désactivé.
- Listener limité à l'interface loopback alors que l'émulateur n'est pas traité comme hôte local par le système.

---

### Étape 3 — Identifier l'adresse réseau de la machine hôte

1. Sur la machine hôte, afficher les interfaces réseau et relever l'adresse IP correspondant au réseau de laboratoire utilisé.
2. Consigner cette adresse sous la forme `<IP_HOTE>` (format : A.B.C.D).

Pourquoi ? Le proxy est hébergé sur la machine hôte. Pour que l'émulateur puisse l'atteindre, une adresse IPv4 valide et accessible depuis l'environnement virtuel est indispensable.

À observer :
- Une adresse IPv4 cohérente avec l'environnement réseau de l'émulateur.
- L'absence de conflits avec d'autres interfaces (VPN, interfaces inactives).

Erreurs fréquentes :
- Utiliser l'adresse IP d'un réseau différent (VPN actif, interface désactivée).
- Confondre adresse IP publique et adresse IP locale.

---

### Étape 4 — Configurer le proxy côté Android Emulator

1. Dans l'émulateur, ouvrir les paramètres Wi-Fi.
2. Sélectionner le réseau Wi-Fi actif et accéder à ses détails.
3. Ouvrir les options avancées et sélectionner Proxy : Manuel.
4. Renseigner :
   - Proxy hostname : `<IP_HOTE>`
   - Proxy port : `<PORT_PROXY>`
5. Enregistrer la configuration.

Pourquoi ? Sans configuration de proxy, le trafic du navigateur est acheminé directement vers Internet et n'apparaît pas dans Burp Suite. La configuration manuelle redirige le flux réseau via le listener.

À observer :
- Le proxy affiché en mode Manuel dans les paramètres.
- L'hôte et le port correspondent exactement aux valeurs relevées aux étapes 2 et 3.

Erreurs fréquentes :
- Port incorrect.
- Champ hostname laissé vide.
- Proxy configuré sur un réseau Wi-Fi différent de celui actuellement utilisé par l'émulateur.

---

### Étape 5 — Premier test : capturer du HTTP (validation de base)

1. Dans l'émulateur, ouvrir le navigateur.
2. Accéder à une cible autorisée dans le cadre du laboratoire.
3. Revenir dans Burp Suite et ouvrir l'onglet HTTP history.
4. Vérifier qu'au moins une requête figure dans l'historique.

Pourquoi ? Cette étape constitue la validation fonctionnelle de la chaîne de capture. Elle doit être réussie avant d'aborder le trafic HTTPS.

À observer :
- Une ligne par requête dans l'historique : méthode (GET/POST), URL, code de statut, taille de la réponse.

Erreurs fréquentes :
- Aucun trafic visible : proxy mal configuré ou listener inaccessible depuis l'émulateur.
- Requêtes bloquées parce que l'interception est restée activée.

Point de vigilance : En l'absence de trafic dans l'historique, reprendre les étapes 2 à 4 et vérifier successivement le listener, l'adresse IP et la configuration proxy dans l'émulateur.

---

### Étape 6 — Lire une requête comme un analyste (sans modification)

1. Dans HTTP history, sélectionner une requête capturée.
2. Ouvrir l'onglet Raw et observer :
   - La méthode HTTP (GET, POST, etc.)
   - Le chemin de la ressource et les éventuels paramètres d'URL
   - Les en-têtes : User-Agent, Accept, Cookie, etc.
3. Ouvrir le panneau Inspector pour une lecture structurée :
   - Query parameters
   - Cookies
   - Headers

Pourquoi ? La compétence fondamentale en sécurité mobile est analytique : comprendre ce qui est transmis, dans quel contexte, et avec quelle signification potentielle. La lecture précède toute action.

À observer :
- Paramètres transmis en clair (sur HTTP).
- Cookies de session : présence ou absence d'attributs de sécurité.

Erreurs fréquentes :
- Se limiter au corps de la réponse en ignorant les en-têtes.
- Qualifier un cookie de "secret" sans en analyser le contexte ni les attributs.

Point de vigilance : Toute observation constitue une preuve. Une preuve sans contexte (version de l'application, environnement, date, cible) n'a pas de valeur exploitable dans un rapport d'audit.

---

### Étape 7 — Interception contrôlée (mode pédagogique)

1. Activer l'interception dans Burp Suite pour une courte séquence uniquement.
2. Dans le navigateur de l'émulateur, rafraîchir une page autorisée.
3. Observer dans Burp Suite que la requête est mise en attente (état suspendu).
4. Désactiver l'interception après observation.

Pourquoi ? Comprendre la différence entre le mode passif (historique) et le mode actif (interception) est essentiel pour saisir le rôle d'un proxy interceptant. Ce lab ne vise pas la manipulation de requêtes.

À observer :
- La distinction entre une capture passive (HTTP history) et un blocage actif (Intercept).

Erreurs fréquentes :
- Laisser l'interception activée et bloquer l'ensemble du trafic sans s'en rendre compte.
- Utiliser l'interception pour modifier des requêtes, ce qui sort du périmètre pédagogique de ce lab.

---

### Étape 8 — HTTPS en laboratoire : principe du certificat CA (sans contournement)

Important : Cette section décrit le principe et les bonnes pratiques de laboratoire. L'installation d'un certificat CA est strictement limitée à l'émulateur et doit être annulée à la fin de la séance.

1. Observer l'interface d'installation de certificat dans l'émulateur.
2. Identifier les types proposés : CA certificate, VPN & app user certificate, Wi-Fi certificate.
3. Comprendre : pour que le navigateur accepte le proxy en HTTPS, un certificat CA de laboratoire peut être requis dans l'environnement de test contrôlé.

Pourquoi ? HTTPS protège le trafic contre l'interception par chiffrement de la couche transport. Un proxy d'analyse doit présenter un certificat reconnu par le client pour déchiffrer et observer ce trafic. En l'absence de certificat de confiance, le navigateur refuse la connexion.

À observer :
- La distinction entre un certificat CA (autorité racine, portée étendue) et les certificats utilisateur ou Wi-Fi.
- Les avertissements système affichés lors de l'ajout d'une autorité de confiance.

Erreurs fréquentes :
- Installer un certificat de laboratoire sur un appareil personnel ou un émulateur destiné à d'autres usages.
- Omettre de retirer le certificat à la fin de la séance.

Point de vigilance : Un certificat CA de laboratoire augmente les capacités d'observation mais réduit le niveau de sécurité de l'environnement. Son usage doit être temporaire, documenté, et strictement limité au périmètre du lab.

---

### Étape 9 — Produire un mini-rapport (preuve + contexte)

Constituer une fiche de trace d'une page comprenant les éléments suivants :

1. Périmètre : environnement émulateur de laboratoire, cible autorisée.

2. Configuration :
   - Version de Burp Suite utilisée
   - `<IP_HOTE>` et `<PORT_PROXY>`
   - Date et heure de la séance

3. Preuves :
   - Capture d'écran de l'historique HTTP (1 à 2 requêtes)
   - Détail d'une requête : en-têtes et URL complète

4. Analyse :
   - Nature des données en transit (paramètres, cookies, tokens éventuels)
   - Risques potentiels identifiés (ex. : jeton d'authentification visible en URL, absence d'en-têtes de sécurité côté client)

5. Recommandations défensives :
   - Réduire les données transmises au strict nécessaire
   - Sécuriser les cookies côté serveur (attributs Secure, HttpOnly, SameSite)
   - Appliquer les bonnes pratiques de développement Android en matière de sécurité réseau

Pourquoi ? La valeur d'un analyste en sécurité se mesure à la qualité de sa documentation, pas au nombre d'outils qu'il maîtrise. Un rapport reproductible permet à une autre personne de refaire le test dans des conditions identiques.

Point de vigilance : Un rapport rigoureux distingue clairement ce qui est observé, ce qui est supposé, et ce qui est recommandé.

---

## Checkpoints de validation

Avant de clore la séance, vérifier que chacun des points suivants est satisfait :

- [ ] Burp Suite capture au moins une requête dans HTTP history.
- [ ] Le proxy listener est actif et ses paramètres sont documentés.
- [ ] Le proxy Android est configuré en mode Manuel avec `<IP_HOTE>:<PORT_PROXY>`.
- [ ] L'interception a été utilisée uniquement à titre de démonstration, puis désactivée.
- [ ] Un rapport court a été produit (preuve + contexte).
- [ ] La procédure de nettoyage de fin de séance est planifiée.

---

## Fin de lab — Nettoyage (hygiène)

Le nettoyage est une étape à part entière, non optionnelle.

1. Désactiver le proxy dans l'émulateur : revenir au mode None ou Proxy off dans les paramètres Wi-Fi.
2. Retirer tout certificat de laboratoire installé sur l'émulateur, si applicable.
3. Fermer le projet Burp Suite et n'archiver que les preuves strictement nécessaires, en veillant à l'absence de données sensibles.

Pourquoi ? Un environnement de laboratoire non nettoyé peut affecter les séances suivantes, introduire des configurations parasites, et présenter un risque résiduel en cas de réutilisation de l'appareil ou de l'émulateur dans un autre contexte.

---

## Avertissement légal et éthique

Ce laboratoire est conçu exclusivement à des fins pédagogiques dans un environnement isolé et autorisé. L'interception de trafic réseau sur des systèmes ou applications sans autorisation explicite est illégale dans la plupart des juridictions. Les techniques présentées ici ne doivent en aucun cas être appliquées en dehors du cadre strict de ce cours.

---

*Laboratoire développé dans le cadre du cours Sécurité des applications mobiles.*

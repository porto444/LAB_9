Environnement utilisé
Système Hôte : Windows (Local)
Émulateur : Android Emulator (Environnement contrôlé et autorisé)
Cible : Application Sieve v1.0
Outils utilisés
Drozer : Framework principal pour l'analyse dynamique et l'exploitation des composants IPC.
ADB (Android Debug Bridge) : Pour la communication avec le terminal et la gestion des ports.
Sieve : Application de test vulnérable (Gestionnaire de mots de passe).
Architecture du Lab
/preuves/
├── /activities/       # Analyse des écrans exposés
├── /services/         # Audit des services d'arrière-plan
├── /receivers/        # Vérification des Broadcast Receivers
├── /providers/        # Tests sur les Content Providers (SQLi/Traversal)
├── /manifest/         # Analyse du fichier AndroidManifest.xml
├── /attacksurface/    # Rapport global de la surface d'attaque
└── /screenshots/      # Captures d'écran de la console et de l'émulateur
Commandes Drozer principales
run app.package.attacksurface com.withsecure.example.sieve
run app.activity.info -a com.withsecure.example.sieve
run app.service.info -a com.withsecure.example.sieve
run app.provider.info -a com.withsecure.example.sieve
run app.provider.query content://...
run scanner.provider.injection
run scanner.provider.traversal
Cartographie des composants exposés
Type	Nom	Exporté	Protection
Activity	MainLoginActivity	Oui	Aucune
Activity	PWList	Oui	Aucune
Service	AuthService	Oui	Aucune
Service	CryptoService	Oui	Aucune
Receiver	ProfileInstallReceiver	Oui	android.permission.DUMP
Provider	DBContentProvider	Oui	Faible
Provider	FileBackupProvider	Oui	Vulnérable
Analyse des Risques
1. Composants Exportés sans Protection
Les activités MainLoginActivity et PWList, ainsi que les services AuthService et CryptoService, sont exportés sans aucune permission (Permission: null). Cela permet à n'importe quelle application malveillante installée sur le même appareil de lancer ces composants directement, contournant potentiellement les écrans d'authentification.

2. Content Providers Vulnérables
Le DBContentProvider expose des données sensibles (mots de passe, clés) via des URIs accessibles. Les tests ont révélé des faiblesses permettant la lecture de données sans autorisation adéquate. Le FileBackupProvider présente un risque élevé de Directory Traversal, permettant potentiellement d'accéder à des fichiers privés de l'application.

3. Broadcast Receivers
Le ProfileInstallReceiver est exporté mais protégé par la permission android.permission.DUMP. Bien que protégé, une application disposant de cette permission pourrait interagir avec ce récepteur.

Captures d'écran
Les preuves visuelles de l'audit sont disponibles dans le dossier /preuves/screenshots/.

Recommandations de sécurité
Principe du moindre privilège : Définir android:exported="false" pour tous les composants qui ne nécessitent pas d'interaction externe.
Permissions de Signature : Utiliser protectionLevel="signature" pour restreindre l'accès aux composants IPC aux seules applications du même développeur.
Sécurisation des Providers : Implémenter des requêtes paramétrées pour prévenir les injections SQL et valider strictement les chemins de fichiers pour empêcher les traversées de répertoire.
Audit du Manifest : Revoir systématiquement les balises <activity>, <service> et <provider> pour éviter toute exposition accidentelle.
Conclusion
L'audit réalisé avec Drozer a mis en évidence une surface d'attaque significative sur l'application Sieve. L'absence de permissions sur des services critiques et la mauvaise configuration des Content Providers posent un risque majeur pour la confidentialité des données utilisateur. Ce lab démontre l'importance d'une configuration rigoureuse des composants IPC dès la phase de développement.

Important

Avertissement légal/éthique
Ce projet est réalisé dans un cadre académique et professionnel. L'utilisation de ces outils et techniques sur des systèmes sans autorisation préalable est illégale. L'objectif est purement éducatif et défensif.

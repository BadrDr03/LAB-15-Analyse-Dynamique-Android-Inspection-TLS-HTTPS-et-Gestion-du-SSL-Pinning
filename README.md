# LAB-15-Analyse-Dynamique-Android-Inspection-TLS-HTTPS-et-Gestion-du-SSL-Pinning

Étape 1 — Installer Frida (PC) et démarrer frida-server (Android)
1.1 Installer Frida côté PC
# Recommandé
python -m pip install --upgrade frida frida-tools

# Vérification
frida --version
python -c "import frida; print(frida.__version__)"
1.2 Préparer ADB et l’appareil
Sur l’appareil: Options développeur → activer « Débogage USB ».
Branchez en USB, acceptez l’empreinte de débogage.
adb devices
Attendu: l’appareil listé « device » (pas « unauthorized »).
1.3 Déployer et lancer frida-server
Identifier l’architecture CPU:
adb shell getprop ro.product.cpu.abi
Télécharger sur https://github.com/frida/frida/releases le frida-server-<version>-android-<arch>.xz correspondant à frida --version.
Décompresser (7‑Zip sous Windows), puis transférer et lancer:
adb push frida-server /data/local/tmp/
adb shell chmod 755 /data/local/tmp/frida-server
adb shell "/data/local/tmp/frida-server -l 0.0.0.0"
Option (selon appareil):
adb forward tcp:27042 tcp:27042
adb forward tcp:27043 tcp:27043
Vérifier:
frida-ps -Uai
Remarque versions: la version client = version frida-server (sinon, erreur de connexion).

![Import OVA](https://github.com/user-attachments/assets/55ce3edb-b986-45b3-bc20-17028ae5e1b9)

---

Étape 2 — Mettre en place le proxy et le certificat CA
2.1 Lancer le proxy sur le PC
Burp: onglet Proxy → Intercept ON/OFF; notez l’adresse/port (ex: 127.0.0.1:8080).
mitmproxy: mitmproxy -p 8080 (par défaut 8080).
2.2 Installer la CA proxy sur l’appareil
Accédez à http://burp ou http://mitm.it depuis le téléphone via le navigateur, téléchargez le certificat, installez-le en tant que « certificat CA utilisateur ».
Sur Android 7+, les apps peuvent ignorer les CA utilisateur si elles utilisent Network Security Config; d’où l’intérêt des hooks TrustManager/Conscrypt.
2.3 Rediriger le trafic de l’appareil vers le proxy
Méthode A: configurer un proxy Wi‑Fi sur le téléphone (même réseau que le PC).
Méthode B (recommandée en USB):
# Rediriger le port local PC 8080 vers le téléphone
adb reverse tcp:8080 tcp:8080
Puis, configurez le proxy de l’app (si possible) ou utilisez un outil de redirection sur l’appareil. En pratique, la méthode Wi‑Fi est plus simple pour un début.
Validation: naviguez vers un site HTTPS depuis le téléphone et vérifiez que le proxy voit les requêtes (vous verrez des erreurs d’SSL tant que le bypass n’est pas actif dans l’app cible).

![Import OVA](https://github.com/user-attachments/assets/7a61ea72-0b24-49ba-bae8-0828c40beb59)

![Import OVA](https://github.com/user-attachments/assets/41ed7f14-115e-4c13-9b16-2e162cdeb5e2)

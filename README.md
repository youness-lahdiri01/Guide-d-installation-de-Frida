# LAB 10 — Guide d’installation de Frida

## Objectif

Ce laboratoire a pour objectif de mettre en place l’environnement Frida afin de réaliser de l’instrumentation dynamique sur une application Android.
Il couvre l’installation du client Frida sur le poste local, le déploiement de frida-server sur un appareil Android, la vérification de la communication ainsi qu’une première injection.

---

## Environnement requis

* Python 3 avec pip
* ADB configuré
* Appareil Android rooté ou émulateur
* Connexion USB ou ADB TCP

---

## Installation de Frida (PC)

```bash
pip install frida frida-tools
```

### Vérification

```bash
frida --version
frida-ps --version
python -c "import frida; print(frida.__version__)"
```

---

## Vérification de l’appareil

```bash
adb devices
```

---

## Installation de frida-server (Android)

### 1. Télécharger depuis GitHub (version identique)

### 2. Décompresser

```bash
xz -d frida-server-*.xz
```

### 3. Transfert vers l’appareil

```bash
adb push frida-server /data/local/tmp/
adb shell chmod +x /data/local/tmp/frida-server
```

---

## Lancement de frida-server

```bash
adb shell
su
/data/local/tmp/frida-server -l 0.0.0.0
```

---

## Vérification de la connexion

```bash
frida-ps -U
frida-ps -Uai
```

---

## Test d’injection

### Script `hello.js`

```javascript
Java.perform(function() {
    console.log("Frida fonctionne !");
});
```

### Injection

```bash
frida -U -f com.example.app -l hello.js
```

---

## Console interactive

```bash
frida -U -n com.example.app
```

---

## Exemple de hook (SharedPreferences)

```javascript
Java.perform(function() {
    var sp = Java.use("android.app.SharedPreferencesImpl");

    sp.getString.overload('java.lang.String', 'java.lang.String').implementation = function(key, def) {
        console.log("Key:", key);
        return this.getString(key, def);
    };
});
```
<img width="858" height="420" alt="Screenshot 2026-04-22 162059" src="https://github.com/user-attachments/assets/a65af137-b128-416f-a6bc-742e0706021a" />

---

## Preuves attendues

```bash
frida --version
frida-ps --version
python -c "import frida; print(frida.__version__)"
adb devices
adb shell /data/local/tmp/frida-server -l 0.0.0.0
frida-ps -Uai
frida -U -f com.example.app -l hello.js
```
<img width="1119" height="273" alt="Screenshot 2026-04-22 162043" src="https://github.com/user-attachments/assets/31dd36e9-182e-4c4a-bafb-0337bcd54a87" />

---

## Dépannage

### Simuler une erreur

```bash
killall frida-server
```

### Vérification

```bash
adb shell ps | grep frida
```

### Relancer

```bash
adb shell
/data/local/tmp/frida-server &
```

---

## Bonnes pratiques

* Utiliser la même version Frida côté client et serveur
* Ne pas exposer frida-server sur un réseau non sécurisé
* Travailler dans un environnement autorisé

---

## Nettoyage

```bash
adb shell rm /data/local/tmp/frida-server
```

---

## Auteur

Youness Lahdiri

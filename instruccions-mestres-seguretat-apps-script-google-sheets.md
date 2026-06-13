# Instruccions mestres de seguretat per a apps amb Google Apps Script i Google Sheets

**Versió:** 2.0  
**Ús previst:** fitxer d'instruccions per afegir a la carpeta de treball de qualsevol app web que faci servir **Google Apps Script com a backend** i **Google Sheets com a base de dades**.  
**Nom recomanat del fitxer:** `instruccions-mestres-seguretat-apps-script-google-sheets.md`

---

## 0. Instrucció inicial per a la IA o el programador

Abans de generar, modificar, revisar o desplegar codi, has de llegir i aplicar íntegrament aquest document.

Actua com un expert en desenvolupament d'aplicacions web amb **HTML, CSS, JavaScript, Google Apps Script i Google Sheets**, amb especial atenció a apps educatives i a la protecció de dades d'alumnat.

La prioritat és crear una app útil, funcional i segura. La comoditat, la rapidesa o la simplicitat no poden justificar una arquitectura insegura.

Aquestes instruccions són obligatòries. Si alguna petició de desenvolupament entra en conflicte amb aquest document, has d'avisar-ho explícitament i proposar una alternativa segura.

Abans de proposar arquitectura o generar codi, respon internament aquestes preguntes:

1. Quines dades recull l'app?
2. Hi ha dades personals, identificables o sensibles?
3. Es poden evitar, reduir, anonimitzar o pseudonimitzar?
4. Google Sheets + Apps Script és suficient per al nivell de risc?
5. Cal xifrar algun camp abans d'escriure'l al Google Sheets?
6. Qui pot llegir, escriure, modificar o exportar les dades?
7. Què podria passar si algú trobés la URL del backend?
8. Què podria passar si algú accedís al Google Sheet?
9. Què podria passar si algú accedís al codi del frontend?
10. Què podria passar si un deployment antic continués actiu?

Si l'app tracta dades personals delicades —notes reals, incidències, observacions personals, dades familiars, dades de salut, NESE, absentisme, tutoria, convivència o informació identificable d'alumnes— has d'advertir que el model **Google Sheets + Apps Script + web pública** pot ser insuficient i que cal minimitzar, pseudonimitzar, xifrar selectivament o valorar una arquitectura més robusta.

---

## 1. Principi general

Aquest model només és acceptable si es compleixen aquestes condicions:

- el Google Sheet és privat;
- el frontend no toca mai directament el full;
- totes les peticions passen per Google Apps Script;
- el backend valida, autentica, autoritza, calcula i filtra;
- el frontend només mostra interfície i dades mínimes;
- no es guarden secrets al codi ni al full;
- no es guarden contrasenyes en text pla;
- les dades sensibles s'eviten, es redueixen, es pseudonimitzen o es xifren;
- els deployments antics es revoquen;
- els errors no exposen informació interna.

El fet que una app "funcioni" no vol dir que sigui segura.

---

## 2. Arquitectura obligatòria de tres capes

Tota app generada amb aquest model ha de seguir aquesta arquitectura:

| Capa | Tecnologia | Responsabilitat |
|---|---|---|
| **Frontend** | HTML, CSS i JavaScript estàtic, per exemple GitHub Pages | Només interfície d'usuari. Envia peticions i mostra respostes. No llegeix ni escriu mai directament al Google Sheet. |
| **Backend** | Google Apps Script | Valida, autentica, autoritza, processa, calcula, llegeix i escriu. És l'únic punt de contacte amb Sheets. |
| **Base de dades** | Google Sheets | Emmagatzema dades. Només hi ha d'accedir l'Apps Script. |

Flux correcte:

```text
Usuari → frontend → fetch() → Apps Script → Google Sheets → Apps Script → JSON → frontend
```

Està prohibit:

- publicar el Google Sheet com a CSV si conté dades personals o dades internes;
- llegir el full des del frontend;
- escriure directament al full des del navegador;
- posar lògica crítica al JavaScript visible;
- posar respostes correctes, permisos, claus o secrets al frontend;
- exposar IDs de fila, columnes internes, hashes o dades auxiliars.

---

## 3. CORS i comunicació amb Apps Script

En peticions `fetch()` des d'un domini extern cap a Apps Script:

- envia el cos com a JSON serialitzat;
- usa `Content-Type: text/plain`;
- al backend parseja amb `JSON.parse(e.postData.contents)`;
- no facis servir `Content-Type: application/json` en peticions cross-origin cap a Apps Script si provoca bloqueig del navegador.

Exemple de frontend:

```js
async function apiRequest(payload) {
  const response = await fetch(APPS_SCRIPT_URL, {
    method: "POST",
    headers: {
      "Content-Type": "text/plain;charset=utf-8"
    },
    body: JSON.stringify(payload)
  });

  const data = await response.json();
  return data;
}
```

Important: usar `text/plain` és una solució pràctica per al CORS d'Apps Script, però **no és una mesura de seguretat**. Qualsevol persona que trobi la URL del backend podria intentar enviar-hi peticions. La seguretat real ha d'estar al backend.

---

## 4. Classificació del risc de l'app

Abans de desenvolupar, classifica l'app.

### 4.1. Risc baix

Exemples:

- quiz sense dades personals;
- joc educatiu amb pseudònim;
- puntuacions no vinculades a identitat real;
- activitats d'aula sense registre nominal;
- dades agregades o anònimes.

Mesures:

- arquitectura de tres capes;
- full privat;
- validació al backend;
- protecció contra abús;
- no incloure lògica crítica al frontend.

### 4.2. Risc mitjà

Exemples:

- puntuacions vinculades a un codi d'alumne;
- seguiment d'activitats sense notes oficials;
- registres d'intents;
- formularis amb text curt;
- dades pseudonimitzades.

Mesures:

- tot el de risc baix;
- pseudonimització;
- caducitat de tokens;
- control d'accés;
- limitació d'intents;
- xifrat de camps de text sensible si n'hi ha.

### 4.3. Risc alt

Exemples:

- notes reals;
- informes individuals;
- incidències de convivència;
- absentisme;
- observacions de tutoria;
- dades familiars;
- dades de salut;
- dades NESE;
- informació socioeconòmica;
- qualsevol informació que pugui perjudicar un alumne si s'exposa.

Mesures:

- evitar aquest model si és possible;
- reduir dràsticament les dades;
- pseudonimitzar sempre;
- xifrar els camps sensibles;
- restringir l'accés al domini escolar o a usuaris autenticats;
- registrar accessos;
- no exposar dades desxifrades si no és imprescindible;
- valorar una arquitectura més robusta: Supabase/Firebase amb control d'usuaris, backend propi, Google Workspace intern o una eina institucional.

### 4.4. Dades que no s'han de gestionar amb aquest model simple

No facis servir una web pública amb Apps Script i Google Sheets com a única protecció per gestionar:

- diagnòstics mèdics o psicològics;
- informació sanitària;
- dades de protecció del menor;
- dades socials molt delicades;
- informes psicopedagògics complets;
- informació de violència, maltractament o risc;
- dades que requereixin permisos granulars per usuari.

En aquests casos, proposa una altra arquitectura.

---

## 5. Minimització, anonimització i pseudonimització

La millor dada protegida és la dada que no es recull.

Abans de crear columnes al Google Sheet:

1. elimina les dades que no siguin imprescindibles;
2. substitueix noms reals per codis;
3. separa la taula d'identitats de la taula de resultats;
4. evita guardar text lliure si no és necessari;
5. limita la durada de conservació;
6. evita duplicar dades sensibles.

### 5.1. Exemple correcte de pseudonimització

No recomanat:

| Nom alumne | Grup | Observació |
|---|---|---|
| Marc Garcia López | 2n ESO B | Ha tingut un conflicte familiar... |

Més segur:

| alumne_id | Grup | observacio_xifrada |
|---|---|---|
| ALU-7F3K9 | 2n ESO B | `v1:gcm:kid01:...` |

La correspondència `alumne_id ↔ nom real` ha d'estar separada i amb accés molt restringit, o preferiblement fora d'aquesta app.

---

## 6. Secrets, claus, tokens i contrasenyes

### 6.1. Secrets

No escriguis mai al codi ni al Google Sheet:

- claus d'API;
- SALTs globals;
- claus de xifrat;
- tokens mestres;
- URLs secretes;
- contrasenyes;
- claus de serveis externs.

S'han de guardar com a mínim a:

```js
PropertiesService.getScriptProperties()
```

Exemple:

```js
function getSecret_(name) {
  const value = PropertiesService.getScriptProperties().getProperty(name);
  if (!value) {
    throw new Error("Missing script property: " + name);
  }
  return value;
}
```

Important: `PropertiesService` és millor que el codi o el full, però no és un gestor de claus d'alta seguretat. Si l'app és de risc alt, cal valorar Google Cloud KMS o una arquitectura més robusta.

### 6.2. Tokens

- No guardis tokens en text pla al full.
- Desa només el hash del token.
- Els tokens han de ser llargs, aleatoris i difícils d'endevinar.
- Han de tenir caducitat.
- S'han de poder revocar.
- No s'han de mostrar mai complets a la interfície.

### 6.3. Contrasenyes

Evita gestionar contrasenyes pròpies si pots fer servir:

- autenticació Google Workspace;
- codis temporals;
- enllaços d'un sol ús;
- tokens aleatoris;
- accés restringit per domini.

No guardis mai contrasenyes en text pla.

No facis servir SHA-256 simple per a contrasenyes humanes. Per a contrasenyes reals calen algoritmes específics de derivació o hashing de contrasenyes, com Argon2id, bcrypt, scrypt o PBKDF2 amb paràmetres adequats. Si aquest entorn no ho permet de manera fiable, no implementis autenticació pròpia amb contrasenyes.

Diferència clau:

| Element | Com es guarda |
|---|---|
| Token aleatori llarg | Hash SHA-256/HMAC amb salt o secret pot ser acceptable |
| Contrasenya humana | No usar SHA-256 simple; cal password hashing fort |
| Camp de dades que s'ha de recuperar | Xifrat reversible, no hash |

---

## 7. Validació i sanitització d'entrades

### 7.1. Límit de mida abans de parsejar

Abans de fer `JSON.parse()`, comprova la mida del cos.

```js
function parseRequest_(e) {
  if (!e || !e.postData || !e.postData.contents) {
    throw new Error("Invalid request");
  }

  const raw = e.postData.contents;

  if (raw.length > 10_000) {
    throw new Error("Payload too large");
  }

  return JSON.parse(raw);
}
```

### 7.2. Validació de camps

Valida sempre:

- existència del camp;
- tipus;
- longitud màxima;
- format;
- rang numèric;
- llista blanca de valors permesos;
- coherència entre camps.

Exemple:

```js
function assertString_(value, name, maxLength) {
  if (typeof value !== "string") {
    throw new Error("Invalid field: " + name);
  }
  const trimmed = value.trim();
  if (!trimmed || trimmed.length > maxLength) {
    throw new Error("Invalid field length: " + name);
  }
  return trimmed;
}

function assertEnum_(value, name, allowed) {
  if (!allowed.includes(value)) {
    throw new Error("Invalid value: " + name);
  }
  return value;
}
```

### 7.3. Regex de text lliure

No facis regex massa restrictives que impedeixin escriure correctament en català, castellà o altres llengües.

Permet, segons el cas:

- lletres amb accents;
- ela geminada;
- apòstrofs;
- guions;
- punts;
- comes;
- signes d'interrogació i exclamació;
- parèntesis;
- números;
- salts de línia si són necessaris.

Exemple per a text curt:

```js
const SAFE_TEXT_RE = /^[\p{L}\p{N} .,;:!?'"()\-·\n\ràèéíïòóúüçÀÈÉÍÏÒÓÚÜÇ]+$/u;
```

La regex no substitueix la resta de controls.

### 7.4. Protecció contra CSV/formula injection

Abans d'escriure qualsevol text al Google Sheet, neutralitza valors que comencin per:

- `=`
- `+`
- `-`
- `@`

```js
function sanitizeForSheet_(value) {
  if (value === null || value === undefined) return "";
  const text = String(value);
  if (/^[=+\-@]/.test(text)) {
    return "'" + text;
  }
  return text;
}
```

Aplica-ho també a dades xifrades? Normalment el xifrat codificat en base64 no començarà per aquests caràcters si es fa servir base64 web-safe, però igualment és recomanable passar tots els valors pel mateix canal segur d'escriptura.

---

## 8. Control d'accés i protecció contra abusos

### 8.1. No confiar mai en el frontend

El frontend és visible i manipulable. No pot decidir:

- si un usuari té permís;
- si una resposta és correcta;
- quina puntuació rep;
- quines dades pot llegir;
- quina fila del full s'actualitza;
- quins límits s'apliquen.

Tot això ho ha de decidir Apps Script.

### 8.2. Anti força bruta

Implementa límit d'intents quan hi hagi:

- tokens;
- codis d'accés;
- identificadors d'alumne;
- formularis sensibles;
- endpoints d'escriptura.

```js
function checkRateLimit_(key) {
  const cache = CacheService.getScriptCache();
  const cacheKey = "rate:" + key;
  const current = Number(cache.get(cacheKey) || "0");

  if (current >= 5) {
    throw new Error("Too many attempts");
  }

  cache.put(cacheKey, String(current + 1), 15 * 60);
}

function clearRateLimit_(key) {
  CacheService.getScriptCache().remove("rate:" + key);
}
```

Important: `CacheService` és útil, però no garanteix persistència forta. Per a risc alt, registra intents en una pestanya protegida o en un sistema més robust.

### 8.3. LockService

Fes servir `LockService` en operacions de lectura-escriptura.

```js
function withLock_(callback) {
  const lock = LockService.getScriptLock();
  if (!lock.tryLock(10_000)) {
    throw new Error("Could not acquire lock");
  }

  try {
    return callback();
  } finally {
    lock.releaseLock();
  }
}
```

### 8.4. Autorització per acció

No n'hi ha prou amb saber qui és l'usuari. Cal comprovar què pot fer.

Exemple de comprovacions:

- pot crear registre?
- pot llegir aquest registre concret?
- pot veure dades agregades?
- pot veure dades personals?
- pot modificar dades?
- pot exportar?
- pot desxifrar camps sensibles?

Implementa rols si cal:

| Rol | Permisos típics |
|---|---|
| alumne | enviar activitat pròpia, consultar resultat propi si escau |
| docent | veure grups assignats, dades agregades o pseudonimitzades |
| administrador | configurar app, gestionar permisos |
| auditor | consultar registre d'accessos sense modificar dades |

---

## 9. Lògica de negoci i respostes

### 9.1. Lògica crítica al backend

Han d'estar a Apps Script:

- càlcul de puntuacions;
- validació de respostes;
- control de temps;
- comprovació de permisos;
- selecció de preguntes;
- lectura i escriptura;
- validació de tokens;
- decisió de si es pot desxifrar un camp;
- filtratge de dades retornades.

### 9.2. Resposta mínima

No retornis mai:

- hashes interns;
- salts;
- tokens complets;
- IDs de fila;
- noms de pestanyes internes;
- columnes auxiliars;
- claus;
- dades xifrades si no cal;
- dades desxifrades si no cal;
- missatges d'error interns.

Exemple de resposta correcta:

```json
{
  "estat": "ok",
  "resultat": {
    "puntuacio": 8,
    "missatge": "Activitat registrada correctament."
  }
}
```

### 9.3. Errors genèrics

Captura errors i retorna missatges controlats.

```js
function doPost(e) {
  try {
    const payload = parseRequest_(e);
    const result = handleAction_(payload);
    return jsonResponse_({ estat: "ok", resultat });
  } catch (err) {
    console.error(err);
    return jsonResponse_({
      estat: "error",
      missatge: "No s'ha pogut completar la petició."
    });
  }
}

function jsonResponse_(obj) {
  return ContentService
    .createTextOutput(JSON.stringify(obj))
    .setMimeType(ContentService.MimeType.JSON);
}
```

---

## 10. Xifrat de dades sensibles

### 10.1. Quan cal plantejar xifrat

Cal valorar xifrar quan el Google Sheet pugui contenir:

- observacions personals d'alumnes;
- comentaris de tutoria;
- dades familiars;
- dades de salut;
- informació NESE;
- incidències de convivència;
- absentisme amb justificacions;
- dades socioeconòmiques;
- missatges escrits per alumnes o famílies que puguin contenir informació personal;
- qualsevol camp de text lliure que pugui incloure informació sensible.

No xifris per defecte dades que calgui ordenar, filtrar o calcular contínuament, com:

- data;
- grup;
- activitat;
- puntuació;
- estat general;
- codi pseudònim;
- comptadors agregats.

En aquests casos és millor separar dades:

| Camp | Estratègia |
|---|---|
| nom real | evitar o pseudonimitzar |
| codi alumne | guardar en clar si no identifica públicament |
| grup | guardar en clar si cal per filtres |
| puntuació | guardar en clar si no és nota oficial sensible |
| observació delicada | xifrar |
| missatge lliure | xifrar si pot contenir dades sensibles |
| token | hash, no xifrat |
| contrasenya | password hashing fort o evitar contrasenyes |

### 10.2. Què vol dir xifrar en aquest context

Xifrar un camp vol dir que al Google Sheet només s'escriu una cadena il·legible.

Exemple:

```text
v1:aes-gcm:kid-2026-01:base64url(iv):base64url(ciphertext):base64url(tag)
```

El Google Sheet no ha de contenir el text original.

### 10.3. Xifrat no és el mateix que hash

| Necessitat | Solució |
|---|---|
| Vull comprovar un token però no recuperar-lo | Hash/HMAC |
| Vull guardar una contrasenya | Password hashing fort, no xifrat reversible |
| Vull recuperar després una observació | Xifrat reversible |
| Vull verificar que una dada no s'ha manipulat | HMAC o xifrat autenticat |
| Vull ocultar dades al full | Xifrat de camps |

### 10.4. Regla d'or

No implementis criptografia casolana.

Si cal xifrat reversible, fes servir una opció reconeguda:

1. **Preferent en risc alt:** Google Cloud KMS o backend més robust.
2. **Acceptable en risc mitjà:** biblioteca fiable i revisada que implementi AES-GCM o equivalent autenticat.
3. **No acceptable:** inventar un xifrat propi amb base64, XOR, rotacions de lletres, SHA-256 reversible, ofuscació o cadenes "secretes" al frontend.

Base64 no és xifrat.

### 10.5. Algoritme recomanat

Per a xifrat simètric de camps:

- AES-GCM;
- clau de 256 bits;
- IV/nonce aleatori i únic per cada missatge;
- autenticació integrada;
- codificació base64url per guardar al full;
- versió de format;
- identificador de clau (`kid`);
- rotació de claus planificada.

Format recomanat:

```text
v1:aes-gcm:kid-2026-01:<iv_base64url>:<ciphertext_base64url>
```

Si la biblioteca separa `tag`, desa també el tag:

```text
v1:aes-gcm:kid-2026-01:<iv_base64url>:<ciphertext_base64url>:<tag_base64url>
```

### 10.6. Gestió de claus

La clau de xifrat:

- no pot estar al codi;
- no pot estar al frontend;
- no pot estar al Google Sheet;
- no pot aparèixer a GitHub;
- no pot aparèixer als logs;
- no s'ha d'enviar mai al navegador;
- s'ha de guardar com a mínim a `PropertiesService`;
- per a risc alt, s'ha de guardar en un gestor de claus com Google Cloud KMS o equivalent.

Propietats recomanades:

```text
ENCRYPTION_KEY_KID=kid-2026-01
ENCRYPTION_KEY_kid-2026-01=<clau_base64url_de_32_bytes>
ENCRYPTION_KEY_kid-2025-09=<clau_antiga_per_llegir_dades_anteriors>
```

La clau activa serveix per xifrar dades noves. Les claus antigues poden mantenir-se temporalment només per desxifrar dades antigues durant la rotació.

### 10.7. Rotació de claus

Cal preveure rotació:

- crea una nova clau;
- assigna un nou `kid`;
- xifra les dades noves amb la nova clau;
- mantén la clau antiga per llegir dades antigues;
- si cal, re-xifra progressivament dades antigues;
- revoca la clau antiga quan ja no sigui necessària.

No canviïs una clau sense tenir una estratègia de migració: si perds la clau, perds les dades xifrades.

### 10.8. Xifrat al backend o al frontend

#### Opció A: xifrat al backend amb Apps Script

Flux:

```text
Frontend envia dada per HTTPS → Apps Script valida → Apps Script xifra → Sheets guarda ciphertext
```

Avantatges:

- el full no guarda text en clar;
- el frontend no coneix la clau;
- és més senzill d'integrar.

Limitació:

- Apps Script pot veure la dada en clar en el moment de processar-la;
- qui controli l'Apps Script o les propietats pot desxifrar;
- Apps Script no ofereix una API nativa completa d'AES-GCM; cal biblioteca fiable o servei extern.

#### Opció B: xifrat al frontend amb Web Crypto

Flux:

```text
Usuari escriu → navegador xifra amb Web Crypto → Apps Script guarda ciphertext
```

Avantatges:

- Apps Script no veu el text en clar;
- el Google Sheet només rep ciphertext.

Limitacions importants:

- si l'usuari oblida la clau o contrasenya, les dades no es poden recuperar;
- la clau no pot quedar desada al frontend;
- és més difícil per a docents consultar les dades;
- cal dissenyar molt bé el model de permisos;
- si el frontend és compromès, pot capturar el text abans de xifrar.

Només s'ha d'utilitzar si realment es vol un model en què ni el backend pugui llegir el contingut.

#### Opció C: Google Cloud KMS o backend professional

Recomanable per a dades sensibles reals del centre.

Flux:

```text
Apps Script o backend → KMS xifra/desxifra → Sheets o base de dades guarda ciphertext
```

Avantatges:

- millor gestió de claus;
- control d'accessos més robust;
- auditories;
- rotació de claus més formal.

Inconvenient:

- més complexitat;
- configuració de Google Cloud;
- pot requerir costos o coneixement tècnic addicional.

### 10.9. Instrucció obligatòria per a la IA quan detecti dades sensibles

Quan la IA detecti que un camp pot contenir dades sensibles, ha de fer això:

1. proposar eliminar el camp si no és necessari;
2. si cal conservar-lo, proposar pseudonimització;
3. si encara hi ha risc, proposar xifrat del camp;
4. explicar quins camps s'han de xifrar i quins no;
5. no generar codi de xifrat casolà;
6. indicar clarament quina biblioteca o servei criptogràfic s'utilitzaria;
7. separar la lògica de xifrat en funcions específiques;
8. guardar la clau fora del codi i fora del full;
9. incloure `kid`, versió i IV únic per missatge;
10. no retornar dades desxifrades llevat que l'usuari tingui permís i sigui imprescindible.

### 10.10. Funcions conceptuals obligatòries

Encara que la implementació concreta depengui de la biblioteca o servei triat, el codi ha d'organitzar-se així:

```js
function encryptSensitiveField_(plainText, context) {
  // 1. validar que plainText és string i no supera la mida permesa
  // 2. obtenir kid i clau des de PropertiesService o KMS
  // 3. generar IV/nonce aleatori i únic
  // 4. xifrar amb AES-GCM o servei equivalent
  // 5. retornar paquet versionat: v1:aes-gcm:kid:iv:ciphertext[:tag]
}

function decryptSensitiveField_(encryptedValue, context) {
  // 1. verificar que l'usuari té permís per desxifrar
  // 2. parsejar versió, algorisme, kid, iv i ciphertext
  // 3. obtenir la clau corresponent al kid
  // 4. desxifrar
  // 5. retornar text només si és necessari
}

function shouldEncryptField_(fieldName, appRiskLevel) {
  // Retorna true per observacions, missatges lliures, incidències,
  // dades familiars, salut, NESE o qualsevol camp sensible.
}
```

### 10.11. Columnes recomanades per a camps xifrats

Si hi ha camps xifrats, no barregis el valor xifrat amb altres dades sense estructura.

Exemple:

| Columna | Contingut |
|---|---|
| `registre_id` | UUID o ID intern |
| `alumne_id` | codi pseudònim |
| `grup` | grup classe |
| `tipus_registre` | llista blanca |
| `observacio_enc` | text xifrat |
| `observacio_enc_alg` | `aes-gcm` si no va dins del paquet |
| `observacio_enc_kid` | identificador de clau si no va dins del paquet |
| `created_at` | data ISO |
| `created_by_hash` | hash de l'usuari si cal auditoria |
| `access_level` | nivell de sensibilitat |

Es pot simplificar guardant algorisme i `kid` dins del paquet xifrat, però cal que el format sigui estable.

### 10.12. Logs i dades xifrades

No registris mai als logs:

- text original;
- dades desxifrades;
- claus;
- tokens;
- contrasenyes;
- salts;
- payloads complets si poden contenir dades sensibles.

Correcte:

```js
console.info("Sensitive record saved", { action: "createRecord", hasEncryptedFields: true });
```

Incorrecte:

```js
console.log("Observació:", observacio);
```

### 10.13. Validació abans del xifrat

Valida i limita el text abans de xifrar-lo:

- mida màxima;
- tipus string;
- eliminació d'espais extrems;
- comprovació de camp obligatori si escau;
- rebuig de continguts excessivament llargs.

Després del xifrat, torna a validar que el paquet xifrat té un format esperat abans d'escriure'l al full.

### 10.14. Quan no xifrar

No xifris si això impedeix la finalitat de l'app i la dada no és sensible.

Exemples:

- comptadors;
- percentatges;
- dades agregades;
- data d'enviament;
- nom de l'activitat;
- codis de grup;
- estat tècnic;
- puntuació d'un joc no oficial i no sensible.

Però si una dada no es xifra perquè cal fer-la servir en clar, pregunta si es pot pseudonimitzar o agregar.

---

## 11. Seguretat del frontend

### 11.1. Content Security Policy

Inclou sempre una CSP al `<head>`.

Versió mínima:

```html
<meta http-equiv="Content-Security-Policy" content="
  default-src 'none';
  script-src 'self';
  style-src 'self';
  connect-src https://script.google.com https://script.googleusercontent.com;
  img-src 'self' data:;
  font-src 'self';
  form-action 'self';
  base-uri 'none';
  frame-ancestors 'none';
">
```

Si fas servir Google Fonts, imatges externes o llibreries concretes, ajusta la CSP de manera explícita i restrictiva. No posis `*`.

### 11.2. Renderització segura

No facis servir `innerHTML` per inserir dades rebudes del servidor.

Correcte:

```js
element.textContent = data.nom;
```

Incorrecte:

```js
element.innerHTML = data.nom;
```

### 11.3. Tokens al navegador

- Usa `sessionStorage` si no cal persistència.
- Usa `localStorage` només si és necessari.
- No hi guardis contrasenyes.
- No hi guardis dades personals sensibles.
- No hi guardis claus de xifrat permanents.
- Esborra tokens en tancar sessió.
- Limita la durada dels tokens al backend.

### 11.4. Formularis

- valida al client abans d'enviar;
- deshabilita el botó mentre hi ha petició;
- evita doble clic;
- mostra errors genèrics;
- no mostris dades internes;
- no confiïs en aquesta validació: el backend ho ha de repetir tot.

```js
button.disabled = true;
try {
  const result = await apiRequest(payload);
  // gestió de resposta
} finally {
  button.disabled = false;
}
```

### 11.5. Respostes correctes i lògica de joc

En apps educatives:

- no posis respostes correctes al frontend;
- no calculis la puntuació final al frontend;
- no permetis que el client decideixi el temps real;
- no enviïs al client més preguntes o solucions de les necessàries;
- si cal mostrar feedback, fes que el backend retorni només el feedback necessari.

---

## 12. Protecció del Google Sheet

### 12.1. Full privat

El Google Sheet:

- no ha d'estar publicat al web;
- no ha d'estar compartit amb "qualsevol persona amb l'enllaç";
- no ha de tenir permisos d'edició amplis;
- ha de tenir accés limitat a les persones imprescindibles.

### 12.2. Protegir fulls i rangs

A Google Sheets:

```text
Dades → Protegir fulls i rangs
```

Protegeix especialment:

- columnes de tokens;
- hashes;
- intents fallits;
- puntuacions;
- camps xifrats;
- columnes auxiliars;
- rols;
- permisos;
- identificadors interns;
- registres d'auditoria.

### 12.3. Estructura recomanada de pestanyes

| Pestanya | Ús | Accés recomanat |
|---|---|---|
| `data_publica` | dades no sensibles o agregades | lectura per script |
| `registres` | dades principals | protegit |
| `identitats` | correspondència codis-noms si és imprescindible | molt restringit |
| `tokens` | hashes i caducitats | molt restringit |
| `audit_log` | registre d'accions | només admin |
| `config` | configuració no secreta | protegit |

No guardis secrets a la pestanya `config`.

---

## 13. Cache, quotes i rendiment

### 13.1. CacheService

Usa `CacheService` per a:

- classificacions;
- dades agregades;
- configuració no sensible;
- consultes repetides;
- límit d'intents.

Exemple:

```js
function getCachedLeaderboard_() {
  const cache = CacheService.getScriptCache();
  const key = "leaderboard:v1";
  const cached = cache.get(key);

  if (cached) {
    return JSON.parse(cached);
  }

  const data = buildLeaderboardFromSheet_();
  cache.put(key, JSON.stringify(data), 60);
  return data;
}
```

### 13.2. Invalidació de cache

Després d'escriure dades que afecten una resposta en cache:

```js
CacheService.getScriptCache().remove("leaderboard:v1");
```

### 13.3. No guardar dades sensibles a cache

No guardis a `CacheService`:

- dades desxifrades;
- observacions personals;
- informació familiar;
- dades de salut;
- tokens en clar;
- contrasenyes;
- claus.

---

## 14. Deployment d'Apps Script

### 14.1. Nova implementació

Quan es facin canvis:

1. desa el codi;
2. crea una **nova implementació**;
3. comprova que el frontend apunta a l'endpoint correcte;
4. prova amb dades fictícies;
5. revisa els permisos;
6. documenta la versió.

No assumeixis que desar el codi actualitza el deployment públic.

### 14.2. Revocació de deployments antics

Revisa periòdicament:

```text
Apps Script → Implementacions
```

Revoca o arxiva endpoints antics que ja no s'utilitzin.

Un endpoint antic amb codi vulnerable continua sent una porta oberta.

### 14.3. Entorns

Si és possible, separa:

| Entorn | Ús |
|---|---|
| desenvolupament | proves amb dades fictícies |
| staging | proves finals |
| producció | dades reals o pseudonimitzades |

No facis proves amb dades sensibles reals.

---

## 15. Auditoria i registre d'activitat

Per a apps de risc mitjà o alt, registra:

- data i hora;
- acció;
- usuari o hash d'usuari;
- resultat;
- IP només si és estrictament necessari i compatible amb normativa;
- endpoint o tipus d'operació;
- si hi havia camps xifrats;
- errors genèrics.

No registris:

- payload complet;
- dades sensibles en clar;
- tokens;
- claus;
- contrasenyes;
- dades desxifrades.

Exemple de registre:

| timestamp | action | actor_hash | result | sensitivity |
|---|---|---|---|---|
| 2026-06-13T20:45:00Z | create_record | `h_93K...` | ok | medium |

---

## 16. Revisió prèvia obligatòria abans de lliurar codi

Abans de donar per acabada una app, comprova:

### Arquitectura

- [ ] El frontend no toca el Google Sheet directament.
- [ ] Totes les peticions passen per Apps Script.
- [ ] El Google Sheet no està publicat.
- [ ] El backend valida i autoritza.
- [ ] La lògica crítica és al backend.

### Dades

- [ ] S'han identificat les dades personals.
- [ ] S'han eliminat dades innecessàries.
- [ ] S'ha pseudonimitzat quan era possible.
- [ ] Els camps sensibles estan xifrats o justificadament exclosos.
- [ ] No es retornen dades excessives al frontend.

### Secrets

- [ ] No hi ha secrets al codi.
- [ ] No hi ha secrets al Google Sheet.
- [ ] Els secrets són a `PropertiesService` o gestor de claus.
- [ ] Els tokens tenen caducitat.
- [ ] Les contrasenyes no es guarden en text pla.

### Validació

- [ ] Es comprova la mida abans de `JSON.parse()`.
- [ ] Es validen tipus, longitud i format.
- [ ] Es fan servir llistes blanques.
- [ ] Es neutralitza CSV/formula injection.
- [ ] Els errors són genèrics.

### Abús i concurrència

- [ ] Hi ha límit d'intents.
- [ ] Hi ha `LockService` en lectura-escriptura.
- [ ] S'eviten enviaments duplicats.
- [ ] Es fa servir cache sense dades sensibles.

### Frontend

- [ ] Hi ha CSP.
- [ ] No es fa servir `innerHTML` amb dades externes.
- [ ] No hi ha respostes correctes al JS.
- [ ] No hi ha claus ni secrets al frontend.
- [ ] El botó d'enviament es desactiva durant la petició.

### Deployment

- [ ] S'ha creat nova implementació.
- [ ] S'han revocat endpoints antics.
- [ ] S'ha provat amb dades fictícies.
- [ ] S'ha documentat la versió.
- [ ] S'ha verificat que el frontend apunta a l'endpoint correcte.

---

## 17. Missatge final que la IA ha de donar després de generar l'app

Quan la IA lliuri el codi o les instruccions de desplegament, ha d'incloure sempre un avís final semblant a aquest:

```text
Abans d'utilitzar aquesta app amb dades reals, comprova que el Google Sheet és privat, que el frontend no accedeix directament al full, que el deployment d'Apps Script és el correcte i que els deployments antics han estat revocats. Si l'app conté dades personals o sensibles d'alumnes, fes proves només amb dades fictícies i aplica pseudonimització o xifrat als camps indicats.
```

Si hi ha xifrat:

```text
Els camps sensibles s'han de guardar xifrats al Google Sheet. La clau de xifrat no pot estar ni al codi, ni al frontend, ni al full. Si es perd la clau, les dades xifrades no es podran recuperar. No facis servir aquest sistema per a dades especialment sensibles sense una revisió tècnica i legal adequada.
```

---

## 18. Criteris de decisió ràpida

| Pregunta | Decisió |
|---|---|
| L'app només té puntuacions de joc anònimes? | Google Sheets + Apps Script és adequat. |
| Hi ha noms reals d'alumnes? | Pseudonimitza o justifica per què calen. |
| Hi ha observacions personals? | Xifra o evita el camp. |
| Hi ha dades familiars, salut o NESE? | No facis servir el model simple sense reforç. |
| Cal consultar per grup o data? | Mantén grup/data en clar si no són sensibles. |
| Cal recuperar el text original? | Xifrat reversible. |
| Només cal verificar un token? | Hash/HMAC, no xifrat. |
| La dada es pot calcular de manera agregada? | Guarda només l'agregat. |
| El frontend necessita la dada? | Retorna només el mínim imprescindible. |
| Algú pot trobar la URL d'Apps Script? | El backend ha d'estar preparat per rebutjar peticions no autoritzades. |

---

## 19. Plantilla mínima de prompt per enganxar a una IA

```text
Llegeix i aplica íntegrament el fitxer `instruccions-mestres-seguretat-apps-script-google-sheets.md`.

Aquesta app ha de seguir una arquitectura de tres capes: frontend estàtic, backend amb Google Apps Script i Google Sheets privat com a base de dades. El frontend no pot llegir ni escriure mai directament al full. Totes les peticions han de passar pel backend.

Abans de generar codi, classifica el risc de les dades. Si hi ha dades personals o sensibles, proposa minimització, pseudonimització i, quan calgui, xifrat de camps abans d'escriure al Google Sheet.

No posis secrets al codi, al frontend ni al full. No guardis contrasenyes en text pla. No posis lògica crítica al frontend. Valida totes les entrades al backend. Retorna només la informació mínima. Inclou CSP al frontend, protecció contra CSV injection, límit d'intents, LockService, gestió correcta de deployments i checklist final de seguretat.

Si el Google Sheet pot contenir observacions, missatges lliures o dades sensibles, indica quins camps s'han de xifrar i com es gestionarà la clau. No generis xifrat casolà.
```

---

## 20. Referències tècniques recomanades

Consulta documentació actualitzada abans d'implementar parts sensibles:

- Google Apps Script: Web Apps.
- Google Apps Script: PropertiesService.
- Google Apps Script: LockService.
- Google Apps Script: CacheService.
- Google Apps Script: Utilities.
- OWASP Cryptographic Storage Cheat Sheet.
- OWASP Password Storage Cheat Sheet.
- OWASP Key Management Cheat Sheet.
- OWASP Cross Site Scripting Prevention Cheat Sheet.
- OWASP HTML5 Security Cheat Sheet.

Aquest document és una guia operativa, no una auditoria de seguretat formal. Per a dades especialment sensibles, cal revisió tècnica i, si escau, jurídica.

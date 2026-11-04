# sitePBV-v1.0 — pachet de personalizare WordPress

Pachetul reunește fișierele furnizate în `sitePBV-09`, cu modificările pentru Site Icon, paginile solicitate, elementele vizuale și măsurile locale de securitate. Directoarele-sursă de la rădăcină sunt păstrate ca copii de dezvoltare; pentru instalare, folosiți conținutul `wp-content/`.

## Ce conține

- tema `primaria-beba-veche` și pluginul PBV existente, păstrate în `wp-content/`;
- fișierul exact `stema-color.jpg` în `wp-content/uploads/` și în directorul de imagini al temei;
- MU-plugin pentru eliminarea generatorului, limitarea enumerării REST a utilizatorilor, dezactivarea XML-RPC, headere de securitate și migrarea idempotentă a paginilor solicitate/iconului;
- reguli Apache în `wp-content/uploads/.htaccess` pentru a refuza cererile PHP din uploads;
- exemple separate de configurare Apache/Nginx și un snippet pentru wp-config.

## Instalarea fișierelor

1. Faceți backup separat al fișierelor și bazei de date existente.
2. Copiați conținutul folderului `wp-content/` peste folderul `wp-content/` al unei instalări WordPress existente. Îmbinați `uploads/` cu fișierele curente; nu ștergeți încărcările existente.
3. Verificați că `wp-content/mu-plugins/pbv-sitepbv-v1.0-hardening.php` și `wp-content/uploads/stema-color.jpg` există pe server.
4. Autentificați-vă ca administrator o dată. La prima încărcare din wp-admin, MU-pluginul creează paginile lipsă și adaugă o secțiune neutră, cu marcatori unici, numai dacă acea secțiune lipsește. Conținutul existent nu este înlocuit. Pagina `Prezentare & Istoric` este creată ca subpagină a paginii `Despre Comună` dacă lipsește. Separat, pluginul PBV creează pagina `Media` la activare și reîncearcă la încărcările WordPress/admin; conținutul existent al paginii nu este înlocuit.
5. MU-pluginul înregistrează imaginea din uploads ca atașament și setează opțiunea WordPress `site_icon` la stema cerută. Dacă există un alt Site Icon, opțiunea este înlocuită, fără ștergerea vechiului atașament. Verificați în Aspect → Personalizare → Identitatea site-ului.
6. Tema afișează iconul Ghișeul.ro și cele două steaguri și pe ecrane înguste. Meniul implicit al temei include `Media`; când un meniu WordPress este atribuit la locația `primary-menu`, pluginul adaugă idempotent un element real `Media` acelui meniu la următoarea încărcare wp-admin (dacă administratorul are dreptul de a edita meniurile). Filtrul de afișare oferă linkul și dacă adăugarea în baza de date nu reușește. Pentru celelalte secțiuni, verificați legăturile în meniul personalizat.

## Texte adăugate

Textele sunt în limba română și evită nume, adrese, programe, date istorice ori alte fapte locale care nu au putut fi confirmate. Pentru Contact, valorile necunoscute sunt marcate explicit ca informații de completat. Secțiunile se adaugă fără a șterge textul existent; administratorul trebuie să verifice și să valideze orice informație mai veche deja prezentă în baza de date.

## Importul XML `pbv.xml`

`import-content/pbv.xml` este o copie a exportului WordPress WXR furnizat, cu programul casieriei corectat la vineri 08:00–12:00. Fișierul nu este importat automat la pornirea site-ului și nu este un backup complet al WordPress. Dacă doriți să importați conținutul, faceți mai întâi backup la baza de date, apoi porniți manual din wp-admin: **Unelte → Import → WordPress**, instalați/rulați importatorul dacă este necesar, încărcați XML-ul și atribuiți autorii. Verificați conținutul și atașamentele importate în site-ul țintă; disponibilitatea fișierelor media din sursă poate afecta importul lor.

## Media și catalogul Google Drive

Pluginul PBV a fost actualizat la 1.4.1. Pagina `Media` este creată idempotent la activare și reîncercată la vizitele administratorului; dacă pagina există și are conținut, acel conținut este păstrat. La testarea unui subfolder Google Drive, managerul folosește ID-ul părintelui din formularul curent, verifică relația directă prin Drive API și leagă dovada de ID-ul părintelui. După schimbarea ID-ului unui părinte, testați din nou subfolderele afectate înainte de salvare.

## Securitate: ce aplică pachetul și ce cere serverul

### Aplicat în fișierele livrate

- MU-pluginul elimină generatorul WordPress din head și din generator filters.
- XML-RPC este dezactivat implicit; endpoint-urile REST `/wp/v2/users` și `/wp/v2/users/{id}` nu sunt expuse vizitatorilor neautentificați.
- Sunt trimise `X-Content-Type-Options: nosniff`, `X-Frame-Options: SAMEORIGIN` și `Referrer-Policy`. HSTS este trimis numai când WordPress detectează HTTPS.
- `wp-content/uploads/.htaccess` blochează cererile către scripturi PHP/PHTML/PHAR pe Apache.
- Tema afișează un favicon fallback din uploads dacă opțiunea WordPress Site Icon lipsește.

Nu am introdus Content-Security-Policy deoarece tema și funcțiile curente folosesc servicii externe (inclusiv Google Drive), iar politica trebuie construită și testată pentru integrările folosite efectiv.

### Necesită configurare pe server sau în wp-admin

- `wp-config.php`, core-ul WordPress, baza de date și fișierele root ale serverului nu sunt incluse în `sitePBV-09`. Integrați `deployment/wp-config-hardening-snippet.php` în configurația reală, fără să înlocuiți secretele; setați și `display_errors=Off` în configurația PHP a hostingului.
- Dacă hostingul permite, mutați `wp-config.php` în afara rădăcinii web și setați permisiuni `400`/`440`. Pentru directoare și fișiere folosiți în mod uzual `755`/`644`, ajustate conform politicii hostingului.
- Îmbinați fragmentul din `deployment/apache-hardening-rules.example.conf` în configurația Apache existentă sau aplicați echivalentul din `deployment/nginx-hardening.example.conf`. Nu înlocuiți direct configurația serverului fără verificare; `.htaccess` nu se aplică pe Nginx.
- Eliminați/redenumiți `readme.html` din rădăcina WordPress pe server; fișierul nu există în materialele furnizate.
- Protecția la încercări repetate de autentificare, 2FA pentru fiecare administrator, backup-urile automate și monitorizarea integrității trebuie configurate în hosting/wp-admin. Nu am instalat un plugin terț și nu am configurat conturi ori servicii externe.
- XML-RPC se dezactivează și în regulile serverului exemplu. Dacă o integrare verificată îl cere, configurați excepția atât în MU-plugin (`PBV_ALLOW_XMLRPC`), cât și la server.

## Limitele pachetului și verificarea live

Arhiva conține proiectul furnizat și fișierele de personalizare, nu WordPress core, baza de date curentă sau `wp-config.php`; copiile vechi găsite separat nu au fost importate deoarece nu reprezintă baza curentă și includ date inițiale nesigure/nevalidate. Nu a fost disponibil PHP sau un runtime WordPress, deci nu s-a putut rula `php -l`, testa migrarea într-o bază reală, valida setarea Site Icon în wp-admin ori verifica afișarea pe un site live. Înainte de publicare, verificați paginile, meniurile, iconul, regulile serverului și integrarea XML-RPC/REST în mediul real.

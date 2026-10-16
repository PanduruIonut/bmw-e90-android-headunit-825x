# Navigație Android BMW E90 (825X_Pro / AC8257): meniuri ascunse și camera de marșarier

Rezumat în română. Detaliile complete (tabele, cod, dovezi) sunt în [README.md](README.md), în engleză.

**E navigația ta?** În *Setări → About Device* apare MCU `TsBMW…`, CAN `YX-BMW…`, HMI `BMW.G5…`.
Se vinde sub multe nume (AUTOTOP, Alps, Tmotec, Junsun…).

## Camera de marșarier: dungi, ecran negru sau imagine alb-negru împărțită

Navigația are **două** intrări video pentru cameră:
- **CAM IN**: semnal analogic **CVBS**;
- **AHD** (mufa cu scris chinezesc): merge doar pe **720p la 25 de cadre** pe profilul BMW.

O cameră **AHD 1080p** pusă pe intrarea AHD dă dungi sau imagine stricată, iar din meniu nu se poate seta
1080p permanent.

**Ce merge:**
- cameră **AHD 720p 25fps** → mufa AHD, *Rear Camera Type* = **Aftermarket Rear Camera (AHD)**;
- cameră care poate scoate **CVBS** → mufa **CAM IN**, *Rear Camera Type* = **Aftermarket Rear Camera**
  (fără AHD/AVM). ✅ Testat: imagine color, rămâne setat și după repornire.

**Camera Excitat NG06064 (eMAG):** tăiați buclele **alb + maro** (capetele izolate separat), lăsați **verdele**
legat, conectați la **CAM IN**, setați **Aftermarket Rear Camera**.

⚠️ Mufa de alimentare a camerei făcea contact prost și se **încingea**. Lipiți firele direct, cu siguranță de 1 A pe plus.
La BMW alimentați camera din lampa de marșarier **printr-un releu** (schema e în README.md).

## Coduri

În setări (*About Device*, rândul HMI) se deschide o căsuță pentru coduri:
- **8878**: Factory Setup → **Backcar Setup** (CamType, oglindire, linii de parcare)
- **8888**: varianta scurtă a meniului de fabrică
- **5678**: logo la pornire
- ⛔ **9090 / 9191**: NU (șterg ID-ul). În Factory Setup nu apăsați *Device ID*, *MCU update*, *CAN*.

## Altele
- Rulează **Android 9**, chiar dacă scrie 11.
- *Setări → General → Sleep mode*: **On** = pornire rapidă (stand-by), **Off** = oprire completă.
- Are **modem 4G cu slot de SIM**: o cartelă de date îi dă internet chiar cu CarPlay wireless pornit.
- După ce folosiți ADB, **opriți-l**: firmware-ul e semnat cu cheile publice de test Android.

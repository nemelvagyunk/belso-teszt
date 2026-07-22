# Éles belső rendszer — tervjegyzet

*A 2026-07-22-i tervezési beszélgetés összefoglalója. A repo maga csak
felület-előtanulmány (teszt); ez a jegyzet az éles rendszerhez gyűjti a
döntéseket és ötleteket. Indulás: majd ha a hely működik.*

## Alapelvek

- **Vékonyan indulni, modulonként nőni** — mindig csak azt megépíteni, ami már fáj;
  a "nagy rendszer" kinő, nem megépül.
- Egy közös gerinc: az **esemény** entitás (naptár), ehhez kapcsolódik jegy,
  pultforgalom, műszak, beszámoló.
- Statikus/egyszerű frontend + kis backend (első jelölt: Supabase — valódi auth,
  közös DB, sor-szintű jogosultság, ingyenes szint).
- Nyers, időbélyeges adatot kérünk minden integrációtól; aggregálni mi aggregálunk.

## Modulok (fázisok a hely mérföldkövei szerint)

**Pályázati szakaszban is hasznos:** meeting-szervező közös tárolóval
(ráérés-rácsok összesítése, "mikor ér rá mindenki", automatikus időpont-javaslat →
Heti meetingjeim + ICS-meghívó); pályázat-munkanézet (radar szűrt találatai
relevancia-pontozással, határidők, ki dolgozik rajta / beadva / nyertünk);
dokumentumtár jogosultsági szintekkel.

**Nyitáskor:** műszakbeosztás (kiírás → jelentkezés a ráérés-rácsból →
véglegesítés → óranyilvántartás; forgalmi adatból tervezve); készletmodul
(recept-alapú fogyás-levonás, standolás: elméleti vs valós, fogyási ütemből
előrejelzés + rendelési jelzés); POS-integráció; napi cashflow-nézet.

**Üzemelés közben:** jegyeladás-integráció (elővételi görbe, esemény-ROI:
jegy + pult-többlet vs költségek; látogatószám-statisztika pályázati
beszámolókhoz); terem-/próbaterem-foglalás; faliújság + teendők.

## Integrációk

- **POS (baráti fejlesztés, több helyen élesben, NTAK-képes):** közös API-spec
  kell — tételes, időbélyeges eladási adat (mikor/mi/mennyi/mennyiért, akár
  óránkénti bontásig), napi zárás összesítő (bevétel, fizetési módok), közös
  termék-azonosítók a recept/készlet-hozzárendeléshez. REST + token vagy webhook
  záráskor. A spec-vázlatot érdemes még a POS fejlesztése közben leegyeztetni.
- **Jegy:** Tixa vs Cooltix még nyitott. Adatkapcsolat-oldalon a Cooltixnak van
  dokumentált API-ja (értékesítési adatok, check-in, saját felületről árusítás,
  sandbox); a Tixánál nyilvános API nem látszik — rákérdezni. Döntési szempont
  még: jutalék, kifizetési ütem, közönség-elérés.

## Értesítések

- 1. lépcső: **személyes naptár-feed (ICS)** műszakokhoz/meetingekhez — a telefon
  natív emlékeztetője szól (pl. 1 órával kezdés előtt), infrastruktúra nélkül.
- 2. lépcső: **PWA + web push** — telepíthető webapp, felhasználónként beállítható
  értesítések: műszak-emlékeztető, műszakcsere/beugrós-riasztás, készlet-jelzés
  az üzletvezetőnek, meeting-értesítő, jegyeladási mérföldkövek.

## Szerepek és biztonság (kritikus — belső szabályzat + pénzügy is lesz benne)

- **Jogosultsági mátrix**, nem csak 2-3 szint: szerepek (tulajdonos, üzletvezető,
  műszakos, programszervező, könyvelő, tag, …) × modulonkénti olvasás/szerkesztés.
  Pénzügy: csak tulajdonos + felhatalmazottak. Bér/műszakadat: mindenki csak a
  sajátját. Szabályzat: mindenki olvas, kevesen szerkesztenek.
- **Minden jogosultság-ellenőrzés szerveroldalon** (a kliensoldali rejtés csak
  kényelmi réteg). Adatok privát adatbázisban — publikus repo kizárva.
- Személyenkénti fiókok (közös login tilos), vezetői szerepekhez **2FA**,
  lejáró munkamenetek, jelszó helyett lehetőleg passkey.
- **Audit-napló** a kényes műveletekről (ki, mikor, mit).
- Rendszeres mentés + visszaállítási terv.
- GDPR: munkatársi személyes/béradat kezelése → adatkezelési szabályzatba felvenni.

## Külső (publikus) oldal — sokkal egyszerűbb, statikus is lehet

- **Bemutatkozó oldal** + **megközelítés** (térkép, tömegközlekedés). Mellé:
  Google Business Profile — a legtöbben a Térképen keresnek majd.
- **Egy eseménynaptár** kategória-szűrővel (koncert / buli / kultúra) és
  heti/havi nézetváltóval — NEM hat külön aloldal. Eseménykártyánként
  jegyvásárlás gomb (Cooltix beágyazható saját felületre). Ugyanaz az
  esemény-gerinc, mint a belső rendszerben: egyszer felvitt program.
- **Hírek = hero-szekció**: a következő 2 kiemelt esemény (fesztivál /
  daytime buli) promózva a főoldal tetején.
- **Teremfoglalás** (rendezvényterem, zenekari próbaterem, kisterem,
  szülinap): induláskor beágyazott kész foglaló. Calendly ingyenes szintje
  csak 1 eseménytípus → több teremhez fizetős, vagy alternatíva: Cal.com
  (nyílt forráskódú, bőkezűbb ingyen). KRITIKUS: a foglalónaptár szinkronban
  legyen a rendezvényekkel (koncert estéjére ne lehessen termet bérelni)!
  Később: saját foglaló a belső naptárra kötve.
- Technika: statikus oldal (a radar/belso-teszt receptje) + beágyazások;
  az eseményadat JSON-ból, amit később a belső rendszer szerkeszt.

## Ovis jel

- **Opt-in dísz, nem kötelező és NEM hitelesítési elem.** Aki szeretné, választ
  jelet (készlet 8-ról bővül, foglalt jel nem választható újra), aki nem, annak
  monogram-avatár. A belépést soha nem akadályozhatja. (A tesztben a belépés
  része volt — az csak játékos próba volt.)

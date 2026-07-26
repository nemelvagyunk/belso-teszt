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

**KONYHA lesz!** Következmények: TEÁOR-ba 5610 (éttermi) kötelező + 5620
(catering — a terembérlős rendezvényeknél saját upsell, nem külsős kérdés);
engedély-oldalon HACCP + élelmiszer-higiénia (NÉBIH) + melegkonyhás működési
kör + NTAK-besorolás — ez a leglassabb engedély-vonal, időben kezdeni!
Készletmodul bővül: alapanyagok, SZAVATOSSÁG-figyelés ("fogyóban" mellé
"lejáróban" jelzés), konyhai receptúrák. POS: étel-tételek, konyha- és
pultbevétel szétválasztva a cashflow-ban. Új bevételi műfaj: brunch /
daytime + konyha kombó a nappali üres órákra.

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

## Próbaterem-piac benchmark (2026-07, két független forrás)

- **Szomszéd próbaterem** (5 terem, 3000-4500 Ft/óra, ablakos): júliusi
  (mélyszezonos) héten 97 foglalt sáv = ~19,4 óra/terem/hét, ~21%
  kihasználtság; foglalás szinte mind 16h után; ~3,7M Ft/terem/év szint.
- **Artisfactory** (5 terem + stúdió, 6000 Ft/óra, PRÉMIUM, ablaktalan,
  nem nagyobb termek): 28M Ft éves árbevétel ≈ 4700 teremóra ≈ 14-18
  óra/terem/hét; ~5M+ Ft/terem/év.
- Konvergencia: a piaci kereslet ~15-19 óra/terem/hét sávban stabil.
  A PRÉMIUM kevesebb órával is többet hoz teremenként → nem alulárazni!
  LOKÁCIÓ-KALIBRÁCIÓ: az Artisfactory prémium lokáció, a miénk "négyes" —
  ezért az árhorgony NEM ő, hanem a közeli szomszéd (azonos vonzáskörzet!):
  szomszéd-ár +10-15% USP-felár = esti listaár 4500-5000 induláskor
  (a demó-árlista pont ez). Az 5500-6000-es Artisfactory-szint felzárkózási
  cél telített naptár + felépült márka után, nem induló ár. A kedvezmény-
  rendszer (last-minute, online, próbázó-korsó) a listaárból lefelé dolgozik.
  4 teremre reális bruttó éves sáv: ~12M (konzervatív) → ~19-22M (prémium).
  GEAR-TÉNYEZŐ: a prémium ár három lába lokáció + hangszerpark + márka —
  ebből a gear az egyetlen azonnal fejleszthető (az Artisfactory ára részben
  a drágább backline). Út: házon belüli árlétra — először EGY terem (D)
  kap prémium backline-t magasabb áron, saját adatból mérjük a fizetési
  hajlandóságot, aztán a bevételből fokozatos gear-fejlesztés a többiben.
- Az ablak NEM értéktényező a piacon (a prémium szereplő is ablaktalan) —
  a mi USP-nk (színpad+fellépés, pult, kódzár 0-24, backline) prémiumot indokol.

## KRITIKUS KORLÁT: albérletbe adás TILOS (fő bérleti szerződés)

- Minden terem-"kiadás" SZOLGÁLTATÁSKÉNT fut, nem bérbeadásként: hozzáférés +
  szolgáltatás (felügyelet, technika, backline, takarítás, kódzár, házirend),
  a birtok nálunk marad. Ingatlan-bérbeadás TEÁOR-t (6820-féle) fel se venni.
- Szerződések neve/tartalma: "teremhasználati / szolgáltatási szerződés",
  SOHA nem "bérleti". Az éles weboldalakon is: "teremhasználat / foglalás",
  nem "terembérlés" (a /berles/ demó átnevezendő élesítéskor!).
- A fő bérleti szerződés pontos tilalmi klauzuláját + a szolgáltatás-modellt
  ügyvéddel átnézetni. Az óradíjas, kódzáras, backline-os modell a
  szolgáltatás-jelleget erősíti — ez jogilag is jól áll.

## Tevékenységek + TEÁOR (könyvelővel véglegesítendő, TEÁOR'25-ben!)

VÉGLEGES LISTA (a megnevezés a mérvadó, a könyvelő TEÁOR'25-ben jelenti be):
5630 Italszolgáltatás (FŐTEVÉKENYSÉG) · 5610 Éttermi vendéglátás ·
5620 Rendezvényi étkeztetés · 9020 Előadó-művészet · 9031 Művészeti
létesítmények és helyszínek működtetése · 9039 Előadó-műv. kiegészítő ·
8230 Konferencia-szervezés · 8211 Összetett irodai szolg. (coworking) ·
9329 M.n.s. szórakoztatás (bulik, gyerekprogram, próbaterem-szolg.) ·
9609 M.n.s. személyi szolg. (esküvőszervezés) · 8551 Sport/szabadidős
képzés · 8552 Kulturális képzés · 8559 M.n.s. egyéb oktatás ·
8560 Oktatást kiegészítő · 9313 Testedzési szolg. (kismama torna,
fitnesz-keretben!) · 5914 Filmvetítés (jogdíj filmenként!).
NEM kerül fel: ingatlan-bérbeadás (albérlet-tilalom!), köznevelési kódok,
gyermekfelügyelet (8891 — engedélyköteles). Gyerekprogram: program
jelleggel, szülői jelenléttel. Felnőttképzési tv.: szervezett képzésnél
bejelentés-kötelezettség lehet — hirdetés előtt könyvelővel egyeztetni.
Elektronikus zene/bulik: zenés-táncos rendezvény bejelentés; szabadtéren
22:00-s zajszabály.
Esküvő (évi 4-6): helyszín+catering a meglévő kódokkal (9031+5620+5630+8230);
ha szervezést is vállalunk → 9609 (m.n.s. egyéb személyi szolgáltatás).
Prémium hétvégi csomagár (1,5-3M Ft/esküvő nagyságrend → évi 8-15M Ft).
Naptár-prioritás: az esküvő egész napos nagyterem+udvar foglalás, fél évre
előre — a koncertprogrammal ütközést a belső naptárnak kell kizárnia.
Udvari zajszabály (22:00) → a buli-rész beltéri folytatása a csomag része.
Gyerek drámatábor (nyár, napközis): kód-oldalon kulturális oktatás + 9329
fedi (szállás nincs → szálláshely-kód sem kell). SZABÁLYOZÁS: napközis tábor
bejelentésköteles (népegészségügy), felügyelő létszám erkölcsi
bizonyítvánnyal, gyermek-közétkeztetési előírások a konyhára — időben
intézni, ez a legpapírosabb program. Üzletileg: nyári völgyszezont tölt,
zárónapi előadás az igazi színpadon (USP), és táborokra sok a pályázat —
érdemes az EGYESÜLET alá szervezni (a kft adja a hátteret), a radar figyeli
a kiírásokat.

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
- **Oldal-család** (egy márka, három hangnem, három célközönség, kereszt-linkekkel):
  1) közösségi/programos oldal (bulis hangulat, próbaterem csak visszafogott
  említés), 2) hivatalos terembérlés-oldal (üzleti stílus, árak, ajánlatkérés),
  3) próbaterem mikro-oldal zenekaroknak (backline, sávok, házirend).
  Demók: /kulso/, /berles/, /proba/ a belso-teszt repóban.
- **Dinamikus árazás (próbaterem, később akár termek):** ALAPELV — a cél az
  ÜRES SÁVOK MEGTÖLTÉSE, nem a népszerűek megdrágítása. Az ár a listaárról
  csak LEFELÉ mozoghat; a prime time (este) listaáron marad mindig.
  Rétegek: 1) sávos alapárak idősáv szerint (a krónikusan üres délelőtt eleve
  olcsóbb), nyilvános ártáblával; 2) last-minute töltés: a mára/holnapra üres
  sáv automatikusan −30%, sáv-riasztás a feliratkozott zenekaroknak;
  3) a foglalási adatokból (belső rendszer méri a kihasználtságot) az derül ki,
  MELYIK sávot kell akciózni — nem az, hogy mit drágítsunk.
  Átlátható, szabály-alapú, kedvezményként kommunikálva (nem surge);
  a havi bérlők fix, védett áron — a törzsgárda bizalma többet ér.
  + **Online fizetve −10%** (a last-minute-tel összeadódik): tehermentesíti a
  pultosokat, lezárja az önkiszolgáló kört (foglalás→fizetés→ajtókód emberi
  kéz nélkül), csökkenti a no-show-t, automatikusan könyvelődik — és a vendég
  úgy érzi, spórolt. Pultnál fizetni lehet, de listaáron.
  Az online −10% a bérlési oldalon is él (kisterem: konferencia, tréning,
  pszichodráma-csoportok; udvar) — a szerződéses nagytermi bérlés kivétel,
  ott egyedi ajánlat megy.
  + **Próbázó-korsó (induló akció):** a próba-sáv mellé az első korsó fix
  kedvezményes áron (pl. 600 Ft) vagy az első kör −25% — fix, "jár neked"
  jellegű gesztus, nem százalék egy korsón (az forintban érezhetetlen).
  Cél: a próba folyjon át pultidőbe ("máshol még hova-üljünk-be gondolkodás
  van, nálunk lemész a pulthoz"). A POS-integrációval automatizálható:
  a foglalórendszer tudja, ki próbált ma → a kedvezmény magától érvényesül.

## Ovis jel

- **Opt-in dísz, nem kötelező és NEM hitelesítési elem.** Aki szeretné, választ
  jelet (készlet 8-ról bővül, foglalt jel nem választható újra), aki nem, annak
  monogram-avatár. A belépést soha nem akadályozhatja. (A tesztben a belépés
  része volt — az csak játékos próba volt.)

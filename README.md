# OBS Lab — odovzdávací systém s AI hodnotením

Statická webová aplikácia (`index.html`) + Vercel serverless funkcie v `api/`, ktoré bezpečne volajú
Anthropic API, ukladajú hodnotenie do databázy a poskytujú učiteľovi tabuľku známok a metodickú
príručku. API kľúč sa nikdy neposiela do prehliadača — žije len na serveri ako premenná prostredia.
Rovnaká architektúra ako Insta360 Lab (kód triedy, tímy aj samostatná práca, Google Drive odkaz,
zmenšovanie obrázkov, ručné známky, CSV export).

## Štruktúra projektu

```
.
├── index.html               # celá aplikácia (frontend, 10 cvičení, tutoriály)
├── OBS_Lab_Ucebnica.pdf     # učebnica (25 strán), cvičenia odkazujú na konkrétne strany
├── img/                     # 11 schém z učebnice (cvNN-K.jpg)
├── api/
│   ├── evaluate.js          # overí odovzdanie, zavolá Anthropic API a výsledok uloží do databázy
│   ├── config.js            # verejne: zoznam žiakov triedy podľa KÓDU TRIEDY
│   ├── teacher.js           # chránené heslom: tabuľka, triedy, kódy, ručné známky, príručka
│   ├── _handbook.js         # text metodickej príručky (servíruje ho len teacher.js po zadaní hesla)
│   └── _db.js               # pomocná vrstva nad Upstash Redis (PROJECT = "obslab")
├── package.json
├── .gitignore
└── README.md
```

## Cvičenia

| Cv. | Téma | Str. v PDF |
|---|---|---|
| 1 | Chroma Key a trojbodové svietenie | 4 |
| 2 | Viackamerový živý strih (Multi-cam) | 6 |
| 3 | Audio manažment, smerovanie a DSP filtre | 8 |
| 4 | Obraz v obraze (PiP), zachytávanie obrazoviek, webinár | 10 |
| 5 | Virtuálne štúdio a digitálny pohyb kamery | 12 |
| 6 | Interaktívna grafika, Lower Thirds, Stinger | 14 |
| 7 | Súbežný záznam a live stream (Output Settings) | 16 |
| 8 | Sieťový prenos videa cez NDI | 18 |
| 9 | OBS ako virtuálna kamera (Zoom/Teams) | 20 |
| 10 | Vzdialení hostia cez VDO.ninja | 22 |

Slovník pojmov je na str. 24, zdroje na str. 25, kritériá hodnotenia na str. 2. Ak PDF premenuješ,
uprav konštantu `BOOK` v `index.html`; ak zmeníš strany, uprav `page:` pri cvičeniach a `BOOK_PAGES`.

**Hodnotiace kritériá** (rovnaké pre všetky cvičenia): technická funkčnosť a stabilita 30 b.,
kľúčovanie/svietenie/kompozícia 25 b., práca s audiom 20 b., protokol a dokumentácia 25 b.
Známka: 86–100 = 1, 70–85 = 2, 56–69 = 3, 40–55 = 4, 0–39 = 5 (funkcia `gradeFor` v `api/evaluate.js`).

## Samostatne alebo v tíme

Každé cvičenie sa dá robiť samostatne alebo v tíme (2 až 4 členovia). Predvolene sú všetky cvičenia
tímové (OBS je tímová produkcia); žiak si režim prepne tlačidlami *Tímové / Individuálne*. V tímovom
režime appka ukáže roly a ich úlohy (pole `plan` v `EX`). Jedno odovzdanie platí za celý tím,
hodnotenie je spoločné; učiteľ môže zadať ručnú známku jednému členovi.

## Kód triedy (žiaci vidia len svoju triedu)

- Každá trieda má 6-znakový **kód** (Učiteľský prehľad → nad tabuľkou, tlačidlá *Kopírovať* a *Nový kód*).
- Žiak ho zadá raz pri odovzdávaní (prehliadač si ho zapamätá, v appke je *Zmeniť triedu*) a uvidí
  zoznam mien **len svojej triedy**. Naraz tak môžu pracovať rôzne ročníky.
- Server pri odovzdaní overí, že kód aj členovia patria do tej istej triedy.
- Staršie triedy dostanú kód automaticky pri prvom otvorení v Učiteľskom prehľade.

## Metodická príručka

V Učiteľskom prehľade (po prihlásení heslom) je tlačidlo **Metodická príručka**: metodika, rubriky
a riešenia 10 cvičení, časté chyby, schémy, slovník, klasifikácia a postup práce so systémom.
Tlačidlo *Tlačiť / uložiť ako PDF* ju otvorí v novom okne na tlač. Text je v `api/_handbook.js`
a servíruje sa len po zadaní hesla učiteľa (nie je verejne dostupný).

## Ako to funguje

- **Povinné pri odovzdaní:** meno zo zoznamu, aspoň jeden súbor (screenshot alebo video), komentár
  (min. 15 znakov) a odkaz na Google Drive. **Selfcheck** je len tvrdenie žiaka — AI ho nepočíta ako dôkaz.
- **Hodnotenie ukladá server.** Body kritérií a známku prepočíta server (kritériá sa orežú na maximum,
  `total` je súčet, známka podľa tabuľky). Bez obrázka je kritérium „kľúčovanie, svietenie, kompozícia“
  obmedzené na 10 z 25 b. (AI videá nevidí a zvuk nepočuje).
- **Opakované odovzdanie:** uchová sa posledných 10 pokusov, do tabuľky ide najlepší.
- **Ak AI zlyhá,** odovzdanie sa zaznamená ako „čaká na ručné hodnotenie“ a známku zadáš ručne.
- **Obrázky** sa v prehliadači zmenšia (dlhšia strana max. 1400 px, JPEG), AI dostane najviac 4.
- **Model:** predvolene `claude-haiku-4-5-20251001`; zmena cez premennú prostredia `EVAL_MODEL` + Redeploy.
- **Viac projektov, jedna databáza:** triedy a žiaci sú spoločné; odovzdania a ručné známky sú oddelené
  podľa projektu (`PROJECT = "obslab"` v `api/_db.js`). Zmazanie triedy zmaže jej známky vo všetkých projektoch.

> Staré odovzdania z predchádzajúcej verzie OBS Lab (uložené len v prehliadačoch, JSON súbory) sa
> do novej databázy neprenášajú. Odovzdania od nasadenia tejto verzie sa ukladajú na server.

## Nasadenie

1. **GitHub:** nahraj `index.html`, `OBS_Lab_Ucebnica.pdf`, `package.json`, `.gitignore`, `README.md`
   a priečinky `api/` a `img/` (Add file → Upload files; priečinky pretiahni celé). Commit do `main`.
2. **Vercel:** Add New → Project → vyber repozitár, Framework Preset **Other**.
3. **Environment Variables** (Production, Preview aj Development), potom Redeploy:
   `ANTHROPIC_API_KEY`, `TEACHER_PASSWORD` a premenné databázy (`KV_REST_API_URL` + `KV_REST_API_TOKEN`
   alebo `UPSTASH_REDIS_REST_URL` + `UPSTASH_REDIS_REST_TOKEN`; Vercel → Storage → Upstash Redis → Connect Project).
   Ak už databázu máš z iného projektu (Insta360, Launchpad…), pripoj tú istú — zoznamy tried sa zdieľajú.
4. **Prvé spustenie:** Učiteľský prehľad → heslo → Nová trieda → názov a zoznam žiakov → Uložiť → daj žiakom kód triedy.
5. Ak sa po pushi neobjaví nový deployment (hláška „GitHub may be delayed“), spravíš malý commit (napr. medzera v README).

Odporúčanie: v [console.anthropic.com](https://console.anthropic.com) nastav pri API kľúči tvrdý limit výdavkov.

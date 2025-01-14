# Global Trust Registry by the PathCheck Foundation

PathCheck's Global Trust Regitry serves as main repository of trusted issuers of COVID Credentials from multiple standards (EU's DCC, DIVOC, Smart Health Cards, Good Health Pass, ICAO VDS, LAC Pass etc). This repository merges different Trust List formats in a main registry and exports the entire key set in several formats and in two main environments: production and testing. 

# Structure

We offer 6 static content formats and 1 dynamic server. Users can simply import the entire database from the JSON or CSV files directly or load records one by one from the NodeJS app. 

The main production repository leaves on [registry.json](https://github.com/Path-Check/trust-registry/blob/main/registry.json). This file is updated every day with keys from the EU DCC Gateway and the VCI directory. ICAO's updates happen monthly. Keys that are not part of any external Trust Framework are updated manually by the team. There is also a registry for test keys [test_registry.json](https://github.com/Path-Check/trust-registry/blob/main/test_registry.json). 

After [registry.json](https://github.com/Path-Check/trust-registry/blob/main/registry.json) is updated, our scripts generate equivalent registries in several formats, including: 
- [did.json](https://github.com/Path-Check/trust-registry/blob/main/did.json): A Signed DID Document referenced by a `DID:WEB` URI and signed by a second DID Document in the [signer](https://github.com/Path-Check/trust-registry/blob/signer/did.json) directory
- [registry_normalized.json](https://github.com/Path-Check/trust-registry/blob/main/registry_normalized.json): A JSON file with only the leaf public keys in a PEM format without headers/footers
- [registry_normalized.csv](https://github.com/Path-Check/trust-registry/blob/main/registry_normalized.csv): A CSV table with only the leaf public keys in a PEM format without headers/footers
- [registry_normalized_jwks.json](https://github.com/Path-Check/trust-registry/blob/main/registry_normalized_jwks.json): A JSON structure with public keys as JWKs and the certificate chains as `x5c` properties inside each key 
- [registry_normalized_jwks.csv](https://github.com/Path-Check/trust-registry/blob/main/registry_normalized_jwks.csv): A CSV table with public keys as [DID:JWKs](https://github.com/quartzjer/did-jwk/blob/main/spec.md) and the certificate chains as `x5c` properties inside each key 

The same structure is created for the  [test_registry.json](https://github.com/Path-Check/trust-registry/blob/main/test_registry.json).

# KeyID Mappings

In order to merge all certificates into one list while correctly representing each specification's key Id resolving needs, the repository makes the following changes: 

- ICAO Seals: keyID is the certificate's issuer `CN` (OID 2.5.4.6) + `#` + the `SubjectKeyIdentifier` (e.g. `BE#0PFBaOWBSJ+lLM1O1/iDtarttAs=`) 
- SmartHealth Cards: keyID is the certificate's issuer `URL` + `#` + the key ID from the JWKs (e.g. `https://spec.smarthealth.cards/examples/issuer#EBKOr72QQDcTBUuVzAzkfBTGew0ZA16GuWty64nS-s`)
- EU DCC: keyID remains the same, first 8 bytes of the SHA-256 fingerprint of the DSC encoded in DER (raw) format  (e.g. `pgM4dDtABSg=`)
- DIVOC: keyID remains the same, the DID of the country's root of trust (e.g. `did:india`)
- CRED: keyID remains the same, the domain name reference for the DNS TXT record (e.g. `1A9.PCF.PW`)

The DID Document ([did.json](https://github.com/Path-Check/trust-registry/blob/main/did.json)) file prefixes all keyIDs with the `DID:WEB` URI of the DID Document controller with the framework being used (e.g. `did:web:raw.githubusercontent.com:Path-Check:trust-registry:main#DCC#mtvkV3qrrDQ=`) 

# Main Data Model

The registry requests/returns entities specified as:
- `governanceFramework`: [`CRED`, `EUDCC`, `SmartHealthCards`, `ICAO`, `DIVOC`]
- `identifier`: The DID identifier of a W3C VC, the public key id of CRED and the EU DCC or the Issuer#kid of the SmartHealth Cards Framework
- `credentialType`: The credential type according to the Standard:
  - CRED: Payload type [`BADGE`, `STATUS`, `PASSKEY`, ...]
  - EU DCC: [`v`,`t`,`r`]
  - SmartHealth Cards: [`https://smarthealth.cards#immunization`, ...]
  - ICAO: [`icao.vacc`, `icao.test`]
  - DIVOC: [`VerifiableCredential`, `ProofOfVaccinationCredential`]
- `entityType`: One of [`issuer`, `verifier`, `trustregistry`]
- `status`: the status of the identifier. 
  - `current`: active
  - `expired`: not renewed after the previous valid registration period
  - `terminated`: voluntary termination by the registered party
  - `revoked`: involuntary termination by the governing authority
- `statusDetail`: Optional free text that expands on the status parameter. Generally, this is a message to show to the user.
- `validFromDT`: Indicates that the Identifier status only starts at the indicated time. It should match the date on the certificate
- `validUntilDT`: Indicates the the Identifier validity ends/ended at this date and time. It should match the date on the certificate 
- `displayName`: i18n Display Names
- `displayLogo`: link to the logo in SVG. 
- `displayURL`: link to the issuers main website

# CSV Formats 

The CSV file has the following collumns, formatted accordingly:

```js
  const FRAMEWORK    = 0 // oneOf[`CRED`, `DCC`, `SHC`, `ICAO`, `DIVOC`]
  const KID          = 1 // 
  const STATUS       = 2 // oneOf[`current`, `expired`, `terminated`, `revoked`]
  const DISPLAY_NAME = 3 // Base64 of the displayName.en UTF-8 string 
  const DISPLAY_LOGO = 4 // Base64 of the displayLogo UTF-8 string
  const VALID_FROM   = 5
  const VALID_UNTIL  = 6
  const PUBLIC_KEY   = 7 // PEM content without -----BEGIN PUBLIC KEY----- and -----END PUBLIC KEY----- or a DID:JWK
  const DISPLAY_URL  = 8 // Base64 of the displayURL UTF-8 string
```

# Supported Issuers: 
- [x] State of California
- [x] State of Colorado (3 keys)
- [x] State of Connecticut
- [x] State of Hawaii
- [x] State of Idaho
- [x] State of Kentucky (2 keys)
- [x] State of Louisiana (3 keys)
- [x] State of Massachusetts
- [x] State of Nevada
<details>
  <summary>Show More</summary>
  
- [x] State of New Jersey
- [x] State of New Mexico
- [x] State of New York
- [x] State of Oklahoma
- [x] State of Utah
- [x] State of Virginia
- [x] Province of Alberta (2 keys)
- [x] Province of British Columbia
- [x] Province of Manitoba
- [x] Province of New Brunswick
- [x] Province of Newfoundland and Labrador
- [x] Province of Nova Scotia
- [x] Province of Ontario
- [x] Province of Prince Edward Island (13 keys)
- [x] Province of Québec - Province of Quebec (5 keys)
- [x] Province of the Northwest Territories
- [x] Province of Yukon (2 keys)
- [x] Government of Albania (2 keys)
- [x] Government of Andorra
- [x] Government of Argentina
- [x] Government of Armenia
- [x] Government of Australia (8 keys)
- [x] Government of Austria (9 keys)
- [x] Government of Bahrain
- [x] Government of Barbados (3 keys)
- [x] Government of Belarus
- [x] Government of Belgium (9 keys)
- [x] Government of Benin (3 keys)
- [x] Government of Botswana (3 keys)
- [x] Government of Brazil
- [x] Government of Bulgaria (4 keys)
- [x] Government of Canada (4 keys)
- [x] Government of Cape Verde
- [x] Government of China (17 keys)
- [x] Government of Colombia (2 keys)
- [x] Government of Croatia (3 keys)
- [x] Government of Cyprus (2 keys)
- [x] Government of Czechia (7 keys)
- [x] Government of Côte d’Ivoire (4 keys)
- [x] Government of Denmark (2 keys)
- [x] Government of Ecuador
- [x] Government of El Salvador
- [x] Government of Estonia (4 keys)
- [x] Government of European Union (2 keys)
- [x] Government of Finland (7 keys)
- [x] Government of France (95 keys)
- [x] Government of Georgia (5 keys)
- [x] Government of Germany (165 keys)
- [x] Government of Ghana
- [x] Government of Greece
- [x] Government of Hungary (9 keys)
- [x] Government of Iceland (7 keys)
- [x] Government of India (2 keys)
- [x] Government of Indonesia (8 keys)
- [x] Government of Iran (2 keys)
- [x] Government of Ireland (5 keys)
- [x] Government of Israel
- [x] Government of Italy (11 keys)
- [x] Government of Jamaica
- [x] Government of Japan (7 keys)
- [x] Government of Jordan
- [x] Government of Kazakhstan (2 keys)
- [x] Government of Kosovo
- [x] Government of Kuwait
- [x] Government of Latvia (11 keys)
- [x] Government of Lebanon (9 keys)
- [x] Government of Liechtenstein (6 keys)
- [x] Government of Lithuania (4 keys)
- [x] Government of Luxembourg (18 keys)
- [x] Government of Madagascar
- [x] Government of Malaysia (3 keys)
- [x] Government of Malta (18 keys)
- [x] Government of Mexico
- [x] Government of Moldova (8 keys)
- [x] Government of Monaco (5 keys)
- [x] Government of Montenegro
- [x] Government of Morocco (4 keys)
- [x] Government of Netherlands (8 keys)
- [x] Government of New Zealand (20 keys)
- [x] Government of Nigeria
- [x] Government of North Macedonia (5 keys)
- [x] Government of Norway (12 keys)
- [x] Government of Oman (2 keys)
- [x] Government of Panama (2 keys)
- [x] Government of Peru
- [x] Government of Philippines (5 keys)
- [x] Government of Poland (6 keys)
- [x] Government of Portugal (3 keys)
- [x] Government of Puerto Rico
- [x] Government of Qatar (5 keys)
- [x] Government of Romania (7 keys)
- [x] Government of Russia (3 keys)
- [x] Government of Rwanda
- [x] Government of San Marino
- [x] Government of Serbia
- [x] Government of Seychelles
- [x] Government of Singapore (16 keys)
- [x] Government of Slovakia (4 keys)
- [x] Government of Slovenia (2 keys)
- [x] Government of South Korea (5 keys)
- [x] Government of Spain (33 keys)
- [x] Government of Sri Lanka
- [x] Government of Sweden (7 keys)
- [x] Government of Switzerland (22 keys)
- [x] Government of Taiwan (2 keys)
- [x] Government of Tanzania (2 keys)
- [x] Government of Thailand (10 keys)
- [x] Government of the Faroe Islands (3 keys)
- [x] Government of the Holy See (Vatican City State)
- [x] Government of the Netherlands (39 keys)
- [x] Government of the United Kingdom (21 keys)
- [x] Government of Togo (2 keys)
- [x] Government of Tunisia (2 keys)
- [x] Government of Turkey (6 keys)
- [x] Government of Turkmenistan (3 keys)
- [x] Government of Uganda (2 keys)
- [x] Government of Ukraine (4 keys)
- [x] Government of United Arab Emirates (7 keys)
- [x] Government of United Kingdom (12 keys)
- [x] Government of United Nations
- [x] Government of United States (4 keys)
- [x] Government of Unknown Region
- [x] Government of Uruguay (2 keys)
- [x] Government of Uzbekistan (4 keys)
- [x] Government of Vietnam
- [x] 4Cyte Pathology
- [x] Access Community Health Network
- [x] AccuSense Health (2 keys)
- [x] AdventHealth
- [x] Adventist Health
- [x] Advocate Aurora Health
- [x] Akron Children's Hospital
- [x] Alaska Tribal Health System
- [x] Albertsons Companies (3 keys)
- [x] Allegheny Health Network
- [x] Allina Health System
- [x] AltaMed
- [x] Altru Health System
- [x] American Medical Center
- [x] American Samoa
- [x] AnMed Health
- [x] Ardent sm Affiliates and Community Connect Practices
- [x] Arizona Department of Health Services
- [x] Arkansas Children's
- [x] Asante
- [x] Ascension
- [x] Aspirus
- [x] Asquam Community Health Collaborative
- [x] Atlantic Health System
- [x] Atrium Health (2 keys)
- [x] Atrius Health
- [x] Aultman
- [x] Austin Regional Clinic
- [x] Ballad Health
- [x] Banner Health
- [x] Baptist Health (2 keys)
- [x] Baptist Health South Florida
- [x] Baptist Healthcare System
- [x] Baptist Memorial Health Care
- [x] Barnabas Health (2 keys)
- [x] Bayhealth Medical Center
- [x] Baylor Medicine
- [x] Baylor Scott & White Health
- [x] Beaumont Health
- [x] Beth Israel Lahey Health
- [x] Beth Israel Lahey Health Mount Auburn Hospital
- [x] Big Y Foods Inc
- [x] Billings Clinic
- [x] BioReference Laboratories (3 keys)
- [x] Bismarck
- [x] BJC HealthCare and Washington University Physicians
- [x] Blanchard Valley Health System
- [x] Boca Raton Regional Hospital
- [x] Bon Secours Health System
- [x] Bon Secours Mercy Health
- [x] Borgess Health Alliance Inc.
- [x] Boston Children's Hospital Employees
- [x] Boston Medical Center
- [x] Boulder Community Health
- [x] Bronson Healthcare Group
- [x] Brookwood Baptist Health
- [x] Broward Health
- [x] Brown and Toland Physicians
- [x] Bryan Health
- [x] Cabell Huntington Hospital and Marshall Health
- [x] Cambridge Health Alliance (3 keys)
- [x] Cape Cod HealthCare
- [x] Cape Fear Valley Health
- [x] Capital Health
- [x] Care New England
- [x] Carle Foundation Hospital
- [x] CarolinaEast Medical Center
- [x] CaroMont Health
- [x] Catholic Health
- [x] Catholic Health System
- [x] Cayman Islands Health Services Authority
- [x] CDR Health
- [x] Cedars-Sinai Health System (2 keys)
- [x] Cedars-Sinai Medical Center
- [x] Centra Health
- [x] CentraCare
- [x] Central Maine Healthcare
- [x] Centura Health
- [x] Cerner Clinic
- [x] Charleston Area Medical Center
- [x] Cheyenne Regional Medical Center
- [x] Children's Hospital & Medical Center
- [x] Children's Hospital Los Angeles
- [x] Children's Mercy Hospital
- [x] Children's National Hospital
- [x] Children's Wisconsin
- [x] Chinese Hospital
- [x] ChristianaCare
- [x] CHRISTUS Health
- [x] City of Hope
- [x] City of Philadelphia
- [x] Clara Barton Hospital
- [x] Cleveland Clinic
- [x] Cleveland Clinic Martin Health
- [x] CNMI Immunization Program
- [x] Columbia St. Mary's
- [x] Columbus Regional Health
- [x] Columbus Regional Healthcare System
- [x] CommonSpirit Health (2 keys)
- [x] Commonwealth of Massachusetts
- [x] Community Health Network (2 keys)
- [x] Community Healthcare System
- [x] Community HealthCare System
- [x] Community Medical Centers
- [x] CommUnityCare Health Centers
- [x] Concord Hospital
- [x] Cone Health
- [x] Conemaugh
- [x] Confluence Health (2 keys)
- [x] Connecticut Children's
- [x] Contra Costa Health Services
- [x] Conway Medical Center
- [x] Cook Children's
- [x] Cook County Community Vaccination Program (8 keys)
- [x] Cook County Health
- [x] Cooper University Healthcare
- [x] Coquille Valley Hospital
- [x] Costco
- [x] Cottage Health
- [x] County of Santa Clara Health System
- [x] Covenant Health
- [x] Covenant Healthcare
- [x] CoxHealth
- [x] Crittenton Hospital Medical Center
- [x] Curative Inc.
- [x] CVS Health (6 keys)
- [x] CyncHealth Nebraska
- [x] Dartmouth-Hitchcock Health (2 keys)
- [x] Deaconess Health System
- [x] Delaware Immunization Program
- [x] Denver Health
- [x] Department of Health, Province of Nunavut
- [x] Detroit Medical Center
- [x] Dignity Health (2 keys)
- [x] District of Columbia Department of Health
- [x] Doctors Care
- [x] Doctors Hospital at Renaissance
- [x] Duke University Health System
- [x] Duly Health and Care (2 keys)
- [x] DuPage Medical Group and Edward-Elmhurst Health
- [x] East Boston Neighborhood Health Center
- [x] East Jefferson General Hospital
- [x] eHealth Saskatchewan (2 keys)
- [x] Einstein Healthcare Network
- [x] Eisenhower Health
- [x] El Camino Hospital
- [x] El Rio Health
- [x] Ellis Medicine
- [x] Emory BLUE Patient Portal
- [x] Emory Healthcare GOLD Patient Portal
- [x] Englewood Hospital and Medical Center
- [x] EPIC Management
- [x] Erlanger Health System
- [x] Eskenazi Health
- [x] Essentia Health
- [x] Express Scripts
- [x] Fairfield Memorial Hospital
- [x] Fairview Health Services
- [x] First Choice Community HealthCare
- [x] FirstHealth of the Carolinas
- [x] Fisher-Titus Medical Center
- [x] Food City
- [x] Franciscan Alliance
- [x] Franciscan Missionaries of Our Lady Health System, Inc.
- [x] Froedtert Health
- [x] Fruth
- [x] FSM Department of Health and Social Affairs
- [x] Garnet Health
- [x] Geisinger
- [x] Genesis Health System
- [x] Genesis Healthcare System (2 keys)
- [x] Georgia Regents Health System
- [x] Giant Eagle Pharmacy
- [x] Global Affairs Canada (12 keys)
- [x] Golden Valley Health Centers
- [x] Grady Health
- [x] Greater Baltimore Medical Center (2 keys)
- [x] Group Health Cooperative - South Central Wisconsin
- [x] Guam Department of Public Health & Social Services
- [x] Gundersen Health System
- [x] GW University Medical Faculty Associates
- [x] Hackensack Meridian Health
- [x] Hamilton Health Care System
- [x] Harps Pharmacy
- [x] Hartford HealthCare (2 keys)
- [x] Hartig Drug Co
- [x] Hattiesburg Clinic and Forrest Health (2 keys)
- [x] Hawaii Pacific Health
- [x] Health Quest
- [x] Health Ventures
- [x] Hennepin Healthcare
- [x] Henry Community Health
- [x] Henry Ford Health System (2 keys)
- [x] Heritage Valley Health System
- [x] Hill Physicians Medical Group
- [x] Hoag Clinic
- [x] Holland Hospital
- [x] HonorHealth
- [x] Hospital for Special Surgery (4 keys)
- [x] Hospital Sisters Health System
- [x] Houston Methodist
- [x] Hudson Physicians, S.C
- [x] Huntsville Hospital Health System
- [x] Hurley Medical Center
- [x] Hy-Vee (2 keys)
- [x] iHealth Labs, Inc
- [x] Illinois Department of Public Health
- [x] Indiana Regional Medical Center
- [x] Indiana University Health
- [x] Inova Health System
- [x] Jackson Health System
- [x] Jefferson Health
- [x] John Muir Health
- [x] Johns Hopkins Medicine
- [x] Kaiser Permanente (26 keys)
- [x] Kaleida Health
- [x] Katherine Shaw Bethea Hospital
- [x] Keck Medicine of USC
- [x] Kelsey-Seybold Clinic
- [x] Kennedy Krieger Institute
- [x] KMART PHARMACY
- [x] Laborant LLC
- [x] Lafayette General Medical Center
- [x] Lahey Health
- [x] Lakeland Regional Health (2 keys)
- [x] Lancaster General Health
- [x] Lawrence Memorial Hospital
- [x] LCMC Health
- [x] Legacy Health
- [x] Lehigh Valley Health Network
- [x] Leon Medical Centers
- [x] Lexington Medical Center
- [x] Licking Memorial Health Systems
- [x] LifeBridge Health
- [x] LifePoint Health
- [x] Logansport Memorial Hospital
- [x] Loma Linda University Health, Riverside University Health System, SAC Health System, and Affiliates (2 keys)
- [x] Los Angeles County Department of Health Services
- [x] Lowell General Hospital
- [x] Loyola Medicine
- [x] Lutheran Health Network MyHealthHome
- [x] Magruder Hospital
- [x] Main Line Health
- [x] MaineHealth
- [x] Marcus Daly Memorial Hospital
- [x] Margaret Mary Health
- [x] Marshfield Clinic Health System
- [x] Mary Washington Healthcare
- [x] Maryland Department of Health
- [x] Mass General Brigham
- [x] Maury Regional Health
- [x] Mayo Clinic
- [x] Med Concierge LA
- [x] Medical Associates Clinic
- [x] Medical University of South Carolina
- [x] MediSys Health Network
- [x] MedStar Health
- [x] Meijer
- [x] Memorial Healthcare System
- [x] Memorial Hermann
- [x] Memorial Hospital
- [x] MemorialCare
- [x] Mercy
- [x] Mercy Health Services (MD)
- [x] Mercy Medical Center
- [x] Mercyhealth
- [x] Meritus Health
- [x] Methodist Health System (2 keys)
- [x] Methodist Hospitals (IN)
- [x] Methodist Le Bonheur Healthcare
- [x] Michigan Medicine
- [x] Middlesex Health
- [x] Midwest Medical Center
- [x] Mineola District Hospital
- [x] Mission Health
- [x] Mississippi State Department of Health
- [x] Missouri Delta Medical Center
- [x] MIT Medical
- [x] Moffitt Cancer Center
- [x] Molina Healthcare
- [x] Monongalia General Hospital
- [x] Monroe Clinic
- [x] Montage Health
- [x] Montefiore Medical Center
- [x] Monument Health
- [x] Mount Sinai Health System
- [x] Mt Sinai - FL
- [x] MultiCare
- [x] Munson Healthcare
- [x] My Ascension Seton
- [x] My Health Kaweah Health
- [x] My VCU Health
- [x] MyHealthHome (4 keys)
- [x] MyIR: AZ DC LA MD MS WA WV
- [x] MyMichigan Health
- [x] MyNCH Health Portal
- [x] Nationwide Children's Hospital
- [x] Nebraska Medicine
- [x] New Hanover Regional Medical Center
- [x] New Jersey Urology
- [x] New York City Health and Hospitals
- [x] New York Consortium
- [x] NewYork-Presbyterian Brooklyn Methodist Hospital
- [x] North Kansas City Hospital and Meritas Health
- [x] North Memorial Health
- [x] North Mississippi Health Services
- [x] NorthBay Healthcare
- [x] Northeast Georgia Health System
- [x] Northern Light Health
- [x] NorthShore University HealthSystem
- [x] Northside Hospital
- [x] Northwest Community Hospital
- [x] Northwestern Medicine
- [x] Norton Healthcare
- [x] Novant Health
- [x] NOVO Health Technology Group
- [x] NSW Health Pathology (2 keys)
- [x] NYU Langone Health System
- [x] OCHIN
- [x] Ochsner Health System
- [x] OhioHealth
- [x] Olathe Health
- [x] Olmsted Medical Center
- [x] On-Site Medical Services
- [x] One Brooklyn Health System
- [x] Optum WA
- [x] OptumCare National
- [x] Oregon Health & Science University
- [x] Oregon Health Authority
- [x] Orlando Health and Bayfront Health St. Petersburg
- [x] OSF HealthCare
- [x] Outpost Healthcare
- [x] Owensboro Health
- [x] Pagosa Springs Medical Center
- [x] Palau Registry
- [x] Palos Health
- [x] Parkland Health (2 keys)
- [x] Parkview Health
- [x] PathCheck Foundation
- [x] PeaceHealth
- [x] Pediatric Physicians' Organization at Children's
- [x] Penn Medicine
- [x] Penn State Health
- [x] Penn State Health St. Joseph
- [x] Pharmaca
- [x] Phelps Health
- [x] Piedmont Healthcare/Affiliates and Shepherd Center
- [x] Plumas District Hospital
- [x] Pomona Valley Hospital Medical Center
- [x] Premier Health
- [x] Premise Health
- [x] Presbyterian Healthcare Services
- [x] Primary Health Medical Group
- [x] Prisma Health
- [x] ProHealth
- [x] ProMedica
- [x] Providence St. Joseph Health (3 keys)
- [x] QuadMed
- [x] Regional Medical Center
- [x] Reid Health
- [x] Reliant Medical Group
- [x] Renown Health
- [x] Rite Aid Pharmacy
- [x] Riverbend Medical Group
- [x] Riverside Health System
- [x] Riverside Healthcare
- [x] Riverside Medical Clinic (2 keys)
- [x] Riverside Medical Group
- [x] RMI Ministry of Health and Human Services
- [x] Roper St. Francis Healthcare
- [x] Royal Jubilee Hospital
- [x] Rush University System for Health
- [x] Rutland Regional Medical Center
- [x] République du Sénégal
- [x] Saint Francis Health System
- [x] Saint Luke's Health System KC
- [x] Salem Health
- [x] Salinas Valley Memorial Healthcare System
- [x] Samaritan Health Services
- [x] San Antonio Regional Hospital
- [x] San Francisco Department of Public Health
- [x] San Joaquin General Hospital
- [x] San Juan Regional Medical Center
- [x] Sanford Health
- [x] Sansum Clinic
- [x] Sarah Bush Lincoln Health Center
- [x] SCL Health
- [x] Scripps Health
- [x] Seattle Children's
- [x] Self Regional Healthcare
- [x] Sentara Healthcare
- [x] Shannon Health
- [x] Silver Cross Hospital
- [x] Singing River Health System
- [x] Skripts
- [x] SolutionHealth
- [x] South Carolina Department of Health and Environmental Control
- [x] South Central Regional Medical Center
- [x] South Georgia Medical Center
- [x] South Shore Health
- [x] Southcoast Health
- [x] Southeast Health
- [x] SoutheastHEALTH
- [x] Southwest General Health Center
- [x] Sparrow Health System
- [x] Spartanburg Regional Healthcare System
- [x] Spectrum Health
- [x] Spectrum Health Lakeland
- [x] SSB Diagnostic
- [x] SSM Health
- [x] St. Elizabeth Healthcare
- [x] St. John's Health
- [x] St. Joseph's Health
- [x] St. Joseph’s Hospital Health Center
- [x] St. Luke's Health System (Idaho)
- [x] St. Luke's Hospital
- [x] St. Luke's University Health Network
- [x] Stanford Health Care
- [x] Stony Brook Business Ventures, LLC
- [x] Stony Brook Medicine
- [x] Stormont-Vail
- [x] Sturdy Memorial Hospital and Sturdy Memorial Associates
- [x] Summit Health
- [x] SUNY Upstate
- [x] Surgery Partners - Nashville
- [x] Sutter Health
- [x] Sydney Local Health District (2 keys)
- [x] Tampa General Hospital and University of South Florida (2 keys)
- [x] Tanner Health System
- [x] Tenet Healthcare Corporation
- [x] Tenet HealthSystem Medical Inc. (4 keys)
- [x] Texas Children's Hospital
- [x] Texas Health Resources
- [x] The Brooklyn Hospital Center
- [x] The Christ Hospital
- [x] The Guthrie Clinic
- [x] The Institute for Family Health
- [x] The Ohio State University Wexner Medical Center
- [x] The Portland Clinic
- [x] The State of Rhode Island
- [x] The University of Texas Health Science Center at Houston
- [x] The University of Texas MD Anderson Cancer Center
- [x] The University of Vermont Medical Center
- [x] ThedaCare, Bellin Health, and Affiliates
- [x] TidalHealth
- [x] Torrance Memorial
- [x] Tower Health
- [x] Treat
- [x] TriHealth
- [x] Trinity Health
- [x] Trinity Health North Dakota
- [x] Trinity Health of New England
- [x] Truman Medical Centers
- [x] Tucson Medical Center
- [x] UC Davis Health
- [x] UC Health (Cincinnati)
- [x] UC Health (San Diego / Irvine / Riverside) and Affiliates (2 keys)
- [x] UCHealth
- [x] UCLA (4 keys)
- [x] UConn Health
- [x] UCSF Health
- [x] UF Health
- [x] UHealth – University of Miami Health System
- [x] UHS of Delaware, Inc - Central
- [x] UI Health
- [x] UK HealthCare
- [x] UMass Memorial Health
- [x] UMC Southern Nevada (2 keys)
- [x] UNC Health Care System (2 keys)
- [x] Union General Hospital
- [x] United Health Services
- [x] United Healthcare Services, Inc.
- [x] United Medical Physicians
- [x] UnityPoint Health
- [x] University Health
- [x] University Health Care System
- [x] University Medical Center
- [x] University Medical Center of El Paso
- [x] University of Alabama Hospital
- [x] University of Chicago Medical Center
- [x] University of Iowa and our Community Connect Hospitals and Clinics: Cass Health, ISH, JCHC, MMC, VBCH and WMC
- [x] University of Kansas
- [x] University of Louisville Physicians
- [x] University of Maryland Medical System
- [x] University of Michigan Health-West
- [x] University of Mississippi Medical Center
- [x] University of Missouri Health Care
- [x] University of New Mexico
- [x] University of Pittsburgh Medical Center (UPMC)
- [x] University of South Alabama Medical Center
- [x] University of Tennessee Medical Center
- [x] University of Texas Medical Branch
- [x] University of Texas Southwestern Medical Center
- [x] University of Utah Health
- [x] UPMC Central PA
- [x] Urology Partners of North Texas
- [x] UT Health San Antonio
- [x] UVA Health
- [x] UW Health
- [x] UW Medicine
- [x] Valley Children's Healthcare
- [x] Valley Medical Center
- [x] Valleywise Health
- [x] Vancouver Clinic
- [x] Vanderbilt University Medical Center
- [x] Vault Health
- [x] VCU Health
- [x] Vidant Health
- [x] Virginia Hospital Center
- [x] WakeMed
- [x] Walgreens
- [x] Walmart
- [x] Washington Hospital Healthcare System
- [x] Washington State Department of Health (2 keys)
- [x] WellSpan Health
- [x] Wellstar
- [x] West Tennessee Healthcare
- [x] Westchester Medical Center Health Network
- [x] Western Connecticut Health Network
- [x] Winona Health Services
- [x] Wood County Hospital
- [x] WVU Medicine
- [x] Wyoming Medical Center
- [x] Yakima Valley Farm Workers Clinic
- [x] Yale New Haven Health System and Yale University
- [x] Yavapai Regional Medical Center
- [x] Yuma Regional Medical Center
</details>

# Live API Server

Check an issuer live on our databases: 
```
https://registry.pathcheck.org/query/issuer?governanceFrameworkURI=SmartHealthCards&identifier=https://myvaccinerecord.cdph.ca.gov/creds%237JvktUpf1_9NPwdM-70FJT3YdyTiSe2IvmVxxgDSRb0&credentialType=https://smarthealth.cards%23immunization
```

# Development Overview

This is a NodeJS + Express app

## Running

Install modules:
`npm install`

To run, do:
`npm run dev`

## Testing 

Check an issuer with the following command

```
curl http://localhost:8000/query/issuer?governanceFrameworkURI=SmartHealthCards&identifier=https://myvaccinerecord.cdph.ca.gov/creds%237JvktUpf1_9NPwdM-70FJT3YdyTiSe2IvmVxxgDSRb0&credentialType=https://smarthealth.cards%23immunization
```

To run all tests, do: 

`npm tests`

## Generating new Version

GitHub Actions generates a new [Release](https://github.com/Path-Check/simple-trust-registry/releases) when npm version is run and pushed to the repo.

```
npm version <version number: x.x.x>
```

## Contributing

[Issues](https://github.com/Path-Check/simple-trust-registry/issues) and [pull requests](https://github.com/Path-Check/simple-trust-registry/pulls) are very welcome! :)


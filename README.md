For English version of the Readme, please [click here.](README_EN.md)

## Projekt Feladat Dokumentáció: Raktárkezelő Szoftver (InventoryManager)
Készítette: Princzinger Krisztián, 2026

Jelenleg folyamatban lévő projekt: **`.NET keretrendszerben fejlesztett C# Készletnyilvántartó alkalmazás`**. Egyetemi beadandó projekt lévén a repozitórium a végső leadási határidőig (május) rejtve marad a kód védelme érdekében. Ezért a képeken és a szövegben lévő funkciók változhatnak a jövőben.


## A program célja és szándékolt működése

A feltöltött program egy grafikus felületű (`Windows Forms`) asztali alkalmazás, amely egy raktár terméktípusainak és maguknak a fizikai raktár-lokációknak a nyilvántartására szolgál. A felhasználó képes új terméktípusokat és raktárakat felvinni a rendszerbe, a meglévőket kilistázni, helyben módosítani és törölni (teljes CRUD funkcionalitás), valamint a teljes adatbázist (`products.json`) menteni és betölteni. A program figyeli a mentetlen módosításokat, és figyelmeztetéssel védi a felhasználót az adatvesztéstől. 


<img src="https://github.com/user-attachments/assets/926ed1e5-6307-469a-b691-f363dc7644b8" />


A program `Form`-okon kívül saját osztályokra épül. Létrehozásra került egy `Product` nevű ősosztály, amely a termékek alapadatait (cikkszám, név, nettó ár, áfakulcs, darabszám, valamint a raktárhoz kapcsoló idegen kulcsot: `LocationId`) tárolja. Az osztály szigorúan követi az egységbezárás elveit: az adattagok `protected` láthatóságúak, elérésük és módosításuk kizárólag nyilvános tulajdonságokon keresztül történik. Az objektumok példányosítását paraméteres konstruktorok biztosítják. Ezen felül bevezetésre került a `Location` osztály, amely a telephelyeket/raktárakat reprezentálja, és saját azonosítóval (`Id`), névvel (`LocationName`), valamint címmel (`LocationAddress`) rendelkezik.
 
A `Product` ősosztályból származik az `ElectronicProduct` utód osztály. Ez az öröklődés révén megkapja az alapvető tulajdonságokat, de kiegészül egy speciális, csak az elektronikai cikkekre jellemző adattaggal (garancia hónapokban). Az utód osztály konstruktora a `base` kulcsszóval hívja meg az ős konstruktorát, és a `ToString()` metódus felülírásával (`override`) kiegészíti a megjelenítést.

A szoftver több ablakból áll, amelyek a `System.Windows.Forms.Form` ősosztályból öröklődnek. Új termék felvitelekor a főablak példányosítja a `ProductWindow` alablakot, és a `ShowDialog()` metódussal, modális ablakként jeleníti meg. Az adatátadás a két ablak között egy nyilvános tulajdonság (`CreatedProduct`) segítségével történik: ha az alablakban a validáció sikeres, az ablak létrehozza az objektumot, beállítja a `DialogResult.OK` értéket, majd bezárul. A főablak ezt az eredményt vizsgálja, és csak ekkor kéri el a létrejött objektumot a tulajdonságon keresztül.


<img src="https://github.com/user-attachments/assets/29f6a248-a445-4c46-9a09-25571996b54f" />


Van keresési és rendezési funkció is: a keresés mezőbe történt felhasználói bevitel pillanatában (a `TextChanged` eseményre reagálva) szűri a memóriában tárolt elemeket. A szoftver a C# beépített LINQ (Language Integrated Query) technológiájának és lambda kifejezéseinek (`Where(p => ...)`) használatával végzi a `List<Product>` gyűjtemény lekérdezését. A keresőmotor kis- és nagybetűkre érzéketlen (`ToLower()`), ürestereket eltávolító (`Trim()`) logikával dolgozik, és párhuzamosan vizsgálja a termékek megnevezését (`ItemName`), valamint cikkszámát (`ItemNumber`). Az egyezést mutató elemekből egy új, szűrt lista jön létre. A keresés mellett a program egy beépített rendezési algoritmust is használ, amellyel a felhasználó a listában szerepeltetett tulajdonságokat ABC szerint sorba rendezheti. Az így előállt szűrt és rendezett lista átadásával a rendszer azonnal frissíti a vizuális megjelenítést  (`RefreshList`).


<img src="https://github.com/user-attachments/assets/19a57d4d-0ef3-4cb4-abc4-c99826826e23" />

 
A felület menürendszerrel és gombokkal operál, amelyek `Click` eseményeihez saját logika tartozik. A rendszer számos esetben használ `MessageBox`-ot figyelmeztetésekre és hibaüzenetekre (pl. hibás adatbevitel). A program fel van készítve a véletlen kilépésre is: a `FormClosing` esemény figyeli, hogy vannak-e mentetlen változások, és egy *Yes/No/Cancel* logikájú dialógussal rákérdez a mentésre.


<img src="https://github.com/user-attachments/assets/956717f4-63ca-431b-9243-3468f9d89a9c" />

 
Az alkalmazás egy `List<Product>` típusú generikus gyűjteményben tárolja az adatokat a memóriában. A Mentés/Betöltés funkciók a `System.Text.Json` beépített könyvtár segítségével hajtanak végre fájlműveleteket a `products.json` fájlon. Mentéskor a szoftver a polimorfizmus megőrzése érdekében `List<object>` típusúvá alakítja a gyűjteményt, majd az egészet egyetlen, ember számára is olvashatóan formázott JSON struktúrává szerializálja, és a `File.WriteAllText` metódussal kiírja a fájlba. Betöltéskor a szoftver egyben beolvassa a szöveges fájl tartalmát, és egy `JsonDocument` segítségével elemzi azt. A kód végigiterál a JSON tömb elemein, és egy egyedi tulajdonság-vizsgálattal (a `WarrantyMonth` kulcs meglétének ellenőrzésével) dinamikusan dönti el az objektum típusát. Ezt követően a megfelelő származtatott vagy alaposztály (`ElectronicProduct` vagy `Product`) szerint deszerializálja a csomópontokat, és hozzáadja őket a memóriában lévő gyűjteményhez.


<img src="https://github.com/user-attachments/assets/413b6f04-c94f-48e7-b138-5d3f8baf272b" />

 
A felvitt vagy betöltött adatok egy `ListBox` vezérlőben jelennek meg, mely a termékek felülírt `ToString()` metódusát használja a formázott kiíráshoz. Ha a felhasználó kijelöl egy elemet a listában, a `SelectedIndexChanged` esemény lefut, a szoftver a kijelölt objektumot visszaalakítja (castolja) a megfelelő típusra, majd a tulajdonságait részletesen betölti a felület jobb oldalán található szövegdobozokba (`TextBox`), külön figyelve a számok ezreselválasztós, olvasható formázására. A rendszer helybeni adatszerkesztést is támogat: az interaktív gombokkal a dobozok írhatóvá válnak, majd a módosítás mentésekor az objektum adatai, valamint a *Read-Only* tulajdonságok (Bruttó ár) is azonnal, dinamikusan frissülnek a képernyőn.

Adatbázis architektúra (v0.1 INDEV)
A jelenlegi fejlesztési fázisban a rendszer az alábbi egy-a-többhöz (1:N) kapcsolatot valósítja meg a Raktárak és a Termékek között. Az architektúra tudatosan dokumentált technikai adósságot (Technical Debt) tartalmaz a `Product` tábla elsődleges kulcsát (`CreatedAt`) illetően. 

Idő hiányában, a beadandó projekt stabilitását tartva szem előtt, ez már csak a - tervezett - v0.2-es verzióban kerül újradolgozásra.

A v0.1-indev adatbázis architektúra:
`
+-----------------------+                 +------------------------+
|       LOCATION        |                 |        PRODUCT         |
|                       |                 |                        |
+-----------------------+                 +------------------------+
| PK: Id (GUID)         | 1             N | PK: CreatedAt (Unsafe) |
|     LocationName      |-----------------| FK: LocationId         |
|     LocationAddress   |                 |     ItemNumber         |
+-----------------------+                 |     ItemName           |
                                          |     NetPrice           |
                                          |     VatRate            |
                                          |     ItemCount          |
                                          |                        |
                                          +------------------------+
                                                      ^
                                                      | (Inheritance)
                                          +------------------------+
                                          |   ELECTRONIC_PRODUCT   |
                                          +------------------------+
                                          |     WarrantyMonth      |
                                          +------------------------+`

A **tervezett** v0.2-indev adatbázis architektúra:
`
+-----------------------+           +------------------------+           +------------------------+
|       LOCATION        |           |    INVENTORY_STOCK     |           |    PRODUCT_CATALOG     |
|             		       	|           |  					                 |           |         			            |
+-----------------------+           +------------------------+           +------------------------+
| PK: Id (GUID)         | 1       N | PK: Id (GUID)          | N       1 | PK: ItemNumber (String)|
|     LocationName      |-----------| FK: LocationId         |-----------|     ItemName           |
|     LocationAddress   |           | FK: ItemNumber         |           |     NetPrice           |
+-----------------------+           |     Quantity           |           |     VatRate            |
                                    |     CreatedAt          |           |	   GrossPrice		        |
                                    +------------------------+           +------------------------+
																					 ^
                                                                                     | 
                                                                                     |
                                                                         +------------------------+
                                                                         |   ELECTRONIC_PRODUCT   |
                                                                         | 					      |
                                                                         +------------------------+
                                                                         | PK/FK: ItemNumber      |
                                                                         |        WarrantyMonth   |
                                                                         +------------------------+`

Korábbi képernyőmentések archívuma
<details>
itemCount bevezetése előtti MainForm
<img src="https://github.com/user-attachments/assets/f624a2fc-319b-490c-bce0-006b1a360eeb" />
 
inline editing első verzió
 
<img src="https://github.com/user-attachments/assets/ee788954-7418-4181-9bf6-0b3121260ceb" />

első "új termék" ablak verzió

![new_product](https://github.com/user-attachments/assets/39779ca0-bbe4-447c-ab92-49be734c7959)

első listbox megjelenítési verzió

<img src="https://github.com/user-attachments/assets/2b488a0b-e015-4c34-96d3-d44850277ad0" />

lista sorbarendezések funckió előtti verzió

![MainFormMainFeatures](https://github.com/user-attachments/assets/848a49e0-a313-437c-8fc4-9e4450a7d3dc)

</details>

Copyright (c) 2026 Princzinger Krisztián. All rights reserved.

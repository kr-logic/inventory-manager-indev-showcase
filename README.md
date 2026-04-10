For English version of the Readme, please [click here.](README_EN.md)

## Projekt Feladat Dokumentáció: Raktárkezelő Szoftver (InventoryManager)
Készítette: Princzinger Krisztián, 2026

Jelenleg folyamatban lévő projekt: **`.NET keretrendszerben fejlesztett C# Készletnyilvántartó alkalmazás`**. Egyetemi beadandó projekt lévén a repozitórium a végső leadási határidőig (május) rejtve marad a kód védelme érdekében. Ezért a képeken és a szövegben lévő funkciók változhatnak a jövőben.


## A program célja és szándékolt működése

A feltöltött program egy grafikus felületű (`Windows Forms`) asztali alkalmazás, amely egy raktár terméktípusainak nyilvántartására szolgál. A felhasználó képes új terméktípusokat felvinni a rendszerbe, a meglévőket kilistázni, helyben módosítani és törölni, valamint a teljes adatbázist (`products.json`) menteni és betölteni. A program figyeli a mentetlen módosításokat, és figyelmeztetéssel védi a felhasználót az adatvesztéstől. 

![MainFormMainFeatures](https://github.com/user-attachments/assets/848a49e0-a313-437c-8fc4-9e4450a7d3dc)


A program `Form`-okon kívül saját osztályokra épül. Létrehozásra került egy `Product` nevű ősosztály, amely a termékek alapadatait (cikkszám, név, nettó ár, áfakulcs, darabszám) tárolja. Az osztály szigorúan követi az egységbezárás elveit: az adattagok `protected` láthatóságúak, elérésük és módosításuk kizárólag nyilvános tulajdonságokon keresztül történik. Az objektumok példányosítását paraméteres konstruktorok biztosítják.
 
A `Product` ősosztályból származik az `ElectronicProduct` utód osztály. Ez az öröklődés révén megkapja az alapvető tulajdonságokat, de kiegészül egy speciális, csak az elektronikai cikkekre jellemző adattaggal (garancia hónapokban). Az utód osztály konstruktora a `base` kulcsszóval hívja meg az ős konstruktorát, és a `ToString()` metódus felülírásával (`override`) kiegészíti a megjelenítést.

A szoftver több ablakból áll, amelyek a `System.Windows.Forms.Form` ősosztályból öröklődnek. Új termék felvitelekor a főablak példányosítja a `ProductWindow` alablakot, és a `ShowDialog()` metódussal, modális ablakként jeleníti meg. Az adatátadás a két ablak között egy nyilvános tulajdonság (`CreatedProduct`) segítségével történik: ha az alablakban a validáció sikeres, az ablak létrehozza az objektumot, beállítja a `DialogResult.OK` értéket, majd bezárul. A főablak ezt az eredményt vizsgálja, és csak ekkor kéri el a létrejött objektumot a tulajdonságon keresztül.

![new_product](https://github.com/user-attachments/assets/39779ca0-bbe4-447c-ab92-49be734c7959)

Van keresési funkció is: a keresés mezőbe történt felhasználói bevitel pillanatában (a `TextChanged` eseményre reagálva) szűri a memóriában tárolt elemeket. A szoftver a C# beépített LINQ (Language Integrated Query) technológiájának és lambda kifejezéseinek (`Where(p => ...)`) használatával végzi a `List<Product>` gyűjtemény lekérdezését. A keresőmotor kis- és nagybetűkre érzéketlen (`ToLower()`), ürestereket eltávolító (`Trim()`) logikával dolgozik, és párhuzamosan vizsgálja a termékek megnevezését (`ItemName`), valamint cikkszámát (`ItemNumber`). Az egyezést mutató elemekből egy új, szűrt lista jön létre, amelynek átadásával a rendszer azonnal frissíti a vizuális megjelenítést (`RefreshList`).

![search](https://github.com/user-attachments/assets/9a0adb89-d1e0-4b54-bab1-fa300c4d3b56)

 
A felület menürendszerrel és gombokkal operál, amelyek `Click` eseményeihez saját logika tartozik. A rendszer számos esetben használ `MessageBox`-ot figyelmeztetésekre és hibaüzenetekre (pl. hibás adatbevitel). A program fel van készítve a véletlen kilépésre is: a `FormClosing` esemény figyeli, hogy vannak-e mentetlen változások, és egy *Yes/No/Cancel* logikájú dialógussal rákérdez a mentésre.

<img width="804" height="482" alt="image" src="https://github.com/user-attachments/assets/1b786bf1-61c7-4beb-845c-0d871d6ccc50" />
 
Az alkalmazás egy `List<Product>` típusú generikus gyűjteményben tárolja az adatokat a memóriában. A Mentés/Betöltés funkciók a `System.Text.Json` beépített könyvtár segítségével hajtanak végre fájlműveleteket a `products.json` fájlon. Mentéskor a szoftver a polimorfizmus megőrzése érdekében `List<object>` típusúvá alakítja a gyűjteményt, majd az egészet egyetlen, ember számára is olvashatóan formázott JSON struktúrává szerializálja, és a `File.WriteAllText` metódussal kiírja a fájlba. Betöltéskor a szoftver egyben beolvassa a szöveges fájl tartalmát, és egy `JsonDocument` segítségével elemzi azt. A kód végigiterál a JSON tömb elemein, és egy egyedi tulajdonság-vizsgálattal (a `WarrantyMonth` kulcs meglétének ellenőrzésével) dinamikusan dönti el az objektum típusát. Ezt követően a megfelelő származtatott vagy alaposztály (`ElectronicProduct` vagy `Product`) szerint deszerializálja a csomópontokat, és hozzáadja őket a memóriában lévő gyűjteményhez.

<img width="356" height="154" alt="image" src="https://github.com/user-attachments/assets/0d51abaa-aeef-4912-b71f-e9f45b5419d1" />

 
A felvitt vagy betöltött adatok egy `ListBox` vezérlőben jelennek meg, mely a termékek felülírt `ToString()` metódusát használja a formázott kiíráshoz. Ha a felhasználó kijelöl egy elemet a listában, a `SelectedIndexChanged` esemény lefut, a szoftver a kijelölt objektumot visszaalakítja (castolja) a megfelelő típusra, majd a tulajdonságait részletesen betölti a felület jobb oldalán található szövegdobozokba (`TextBox`), külön figyelve a számok ezreselválasztós, olvasható formázására. A rendszer helybeni adatszerkesztést is támogat: az interaktív gombokkal a dobozok írhatóvá válnak, majd a módosítás mentésekor az objektum adatai, valamint a *Read-Only* tulajdonságok (Bruttó ár) is azonnal, dinamikusan frissülnek a képernyőn.


Korábbi képernyőmentések archívuma
<details>
itemCount bevezetése előtti MainForm
<img width="806" height="486" alt="image" src="https://github.com/user-attachments/assets/f624a2fc-319b-490c-bce0-006b1a360eeb" />
 
inline editing első verzió
 
<img width="806" height="486" alt="image" src="https://github.com/user-attachments/assets/ee788954-7418-4181-9bf6-0b3121260ceb" />

első listbox megjelenítési verzió

<img width="806" height="486" alt="image" src="https://github.com/user-attachments/assets/2b488a0b-e015-4c34-96d3-d44850277ad0" />

</details>

Copyright (c) 2026 Princzinger Krisztián. All rights reserved.

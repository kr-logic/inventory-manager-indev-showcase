## A program célja és szándékolt működése

A feltöltött program egy grafikus felületű (`Windows Forms`) asztali alkalmazás, amely egy bolt terméktípusainak nyilvántartására szolgál. A felhasználó képes új terméktípusokat felvinni a rendszerbe, a meglévőket kilistázni, helyben módosítani és törölni, valamint a teljes adatbázist (`products.csv`) menteni és betölteni. A program figyeli a mentetlen módosításokat, és figyelmeztetéssel védi a felhasználót az adatvesztéstől. 

<img width="806" height="486" alt="image" src="https://github.com/user-attachments/assets/f624a2fc-319b-490c-bce0-006b1a360eeb" />

 
A program `Form`-okon kívül saját osztályokra épül. Létrehozásra került egy `Product` nevű ősosztály, amely a termékek alapadatait (cikkszám, név, nettó ár, áfakulcs) tárolja. Az osztály szigorúan követi az egységbezárás elveit: az adattagok `protected` láthatóságúak, elérésük és módosításuk kizárólag nyilvános tulajdonságokon keresztül történik. Az objektumok példányosítását paraméteres konstruktorok biztosítják.
 
A `Product` ősosztályból származik az `ElectronicProduct` utód osztály. Ez az öröklődés révén megkapja az alapvető tulajdonságokat, de kiegészül egy speciális, csak az elektronikai cikkekre jellemző adattaggal (garancia hónapokban). Az utód osztály konstruktora a `base` kulcsszóval hívja meg az ős konstruktorát, és a `ToString()` metódus felülírásával (`override`) kiegészíti a megjelenítést.

A szoftver több ablakból áll, amelyek a `System.Windows.Forms.Form` ősosztályból öröklődnek. Új termék felvitelekor a főablak példányosítja a `ProductWindow` alablakot, és a `ShowDialog()` metódussal, modális ablakként jeleníti meg. Az adatátadás a két ablak között egy nyilvános tulajdonság (`CreatedProduct`) segítségével történik: ha az alablakban a validáció sikeres, az ablak létrehozza az objektumot, beállítja a `DialogResult.OK` értéket, majd bezárul. A főablak ezt az eredményt vizsgálja, és csak ekkor kéri el a létrejött objektumot a tulajdonságon keresztül.

 
A felület menürendszerrel és gombokkal operál, amelyek `Click` eseményeihez saját logika tartozik. A rendszer számos esetben használ `MessageBox`-ot figyelmeztetésekre és hibaüzenetekre (pl. hibás adatbevitel). A program fel van készítve a véletlen kilépésre is: a `FormClosing` esemény figyeli, hogy vannak-e mentetlen változások, és egy *Yes/No/Cancel* logikájú dialógussal rákérdez a mentésre.

<img width="804" height="482" alt="image" src="https://github.com/user-attachments/assets/1b786bf1-61c7-4beb-845c-0d871d6ccc50" />

 
Az alkalmazás egy `List<Product>` típusú generikus gyűjteményben tárolja az adatokat a memóriában. A Mentés/Betöltés funkciók fájlműveleteket hajtanak végre a `products.csv` fájlon. Mentéskor a szoftver végigiterál a fájl sorain, és objektumtípustól függően (az `is` operátorral vizsgálva) pontosvesszővel elválasztva (`;`) írja ki az attribútumokat egy-egy új sorba. Betöltéskor a `StreamReader` soronként olvassa a fájlt, a `Split` metódussal szétdarabolja azokat, majd a sor elején lévő azonosító (`Normal`/`Electronic`) alapján a megfelelő osztályt példányosítja, és hozzáadja a gyűjteményhez.

<img width="356" height="154" alt="image" src="https://github.com/user-attachments/assets/0d51abaa-aeef-4912-b71f-e9f45b5419d1" />

 
A felvitt vagy betöltött adatok egy `ListBox` vezérlőben jelennek meg, mely a termékek felülírt `ToString()` metódusát használja a formázott kiíráshoz. Ha a felhasználó kijelöl egy elemet a listában, a `SelectedIndexChanged` esemény lefut, a szoftver a kijelölt objektumot visszaalakítja (castolja) a megfelelő típusra, majd a tulajdonságait részletesen betölti a felület jobb oldalán található szövegdobozokba (`TextBox`), külön figyelve a számok ezreselválasztós, olvasható formázására. A rendszer helybeni adatszerkesztést is támogat: az interaktív gombokkal a dobozok írhatóvá válnak, majd a módosítás mentésekor az objektum adatai, valamint a *Read-Only* tulajdonságok (Bruttó ár) is azonnal, dinamikusan frissülnek a képernyőn.

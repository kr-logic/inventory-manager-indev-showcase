A magyar Readme fájl megtekintéséhez [kattints ide.](README.md)

## Project Task Documentation: Inventory Management Software (InventoryManager)
Created by: Princzinger Krisztián, 2026

Currently ongoing project: A **`C# Inventory Management application developed in the .NET framework`**. As a university assignment project, the repository will remain private until the final submission deadline (May) to protect the code. Therefore, the features shown in the images and described in the text may change in the future.

## Purpose and intended operation of the program

The uploaded program is a graphical user interface (`Windows Forms`) desktop application designed for keeping records of product types in a warehouse. The user is able to add new product types to the system, list existing ones, modify and delete them in place, as well as save and load the entire database (`products.json`). The program monitors unsaved modifications and protects the user from data loss with a warning.

![MainFormMainFeatures](https://github.com/user-attachments/assets/848a49e0-a313-437c-8fc4-9e4450a7d3dc)


Besides `Form`s, the program is built on its own classes. A base class named `Product` was created to store the basic data of the products (item number, name, net price, VAT rate, quantity). The class strictly follows the principles of encapsulation: data members have `protected` visibility, and their access and modification are handled exclusively through public properties. Object instantiation is provided by parameterized constructors.

The `ElectronicProduct` derived class inherits from the `Product` base class. Through this inheritance, it receives the basic properties but is supplemented with a special data member specific only to electronic items (warranty in months). The constructor of the derived class calls the base constructor using the `base` keyword, and it extends the display by overriding the `ToString()` method.

The software consists of multiple windows that inherit from the `System.Windows.Forms.Form` base class. When adding a new product, the main window instantiates the `ProductWindow` sub-window and displays it as a modal window using the `ShowDialog()` method. Data transfer between the two windows is achieved using a public property (`CreatedProduct`): if the validation in the sub-window is successful, the window creates the object, sets the `DialogResult.OK` value, and then closes. The main window checks this result and only then retrieves the created object through the property.

![new_product](https://github.com/user-attachments/assets/39779ca0-bbe4-447c-ab92-49be734c7959)


There is also a search function: it filters the items stored in memory in real-time as the user types into the search field (reacting to the `TextChanged` event). The software queries the `List<Product>` collection using C#'s built-in LINQ (Language Integrated Query) technology and lambda expressions (`Where(p => ...)`). The search engine operates with case-insensitive (`ToLower()`) and whitespace-removing (`Trim()`) logic, simultaneously examining both the product name (`ItemName`) and the item number (`ItemNumber`). A new, filtered list is created from the matching elements, which is then passed to immediately update the visual display (`RefreshList`).

![search](https://github.com/user-attachments/assets/496f51c0-427f-479f-a1f8-f554fba62711)



The interface operates with a menu system and buttons, with custom logic attached to their `Click` events. The system frequently uses `MessageBox` for warnings and error messages (e.g., invalid data entry). The program is also prepared for accidental exits: the `FormClosing` event monitors for unsaved changes and prompts the user to save using a *Yes/No/Cancel* logic dialog.

<img width="804" height="482" alt="image" src="https://github.com/user-attachments/assets/1b786bf1-61c7-4beb-845c-0d871d6ccc50" />


The application stores data in memory using a generic `List<Product>` collection. The Save/Load functions perform file operations on the `products.json` file using the built-in `System.Text.Json` library. During the save process, to preserve polymorphism, the software converts the collection to a `List<object>` type, serializes the entirety into a single, human-readable formatted JSON structure, and writes it to the file using the `File.WriteAllText` method. During loading, the software reads the entire content of the text file and parses it using a `JsonDocument`. The code iterates through the elements of the JSON array and dynamically determines the object type via a unique property check (verifying the presence of the `WarrantyMonth` key). Subsequently, it deserializes the nodes according to the appropriate derived or base class (`ElectronicProduct` or `Product`) and adds them to the in-memory collection.

<img width="356" height="154" alt="image" src="https://github.com/user-attachments/assets/0d51abaa-aeef-4912-b71f-e9f45b5419d1" />


The entered or loaded data is displayed in a `ListBox` control, which uses the overridden `ToString()` method of the products for formatted output. When the user selects an item in the list, the `SelectedIndexChanged` event is triggered, the software casts the selected object to the appropriate type, and then loads its properties in detail into the text boxes (`TextBox`) on the right side of the interface, paying special attention to the readable formatting of numbers with thousand separators. The system also supports inline data editing: interactive buttons make the boxes editable, and upon saving the changes, the object's data, as well as the *Read-Only* properties (Gross price), are immediately and dynamically updated on the screen.

Archive of previous screenshots
<details>
MainForm before the introduction of itemCount
<img rc="https://github.com/user-attachments/assets/f624a2fc-319b-490c-bce0-006b1a360eeb" />
  
first version of inline editing

<img src="https://github.com/user-attachments/assets/ee788954-7418-4181-9bf6-0b3121260ceb" />

first version of listbox view 

<img src="https://github.com/user-attachments/assets/2b488a0b-e015-4c34-96d3-d44850277ad0" />

</details>

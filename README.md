# TODO

### PRIORITARIO
~~1. Provare ad implementare la CustomScrollArea (presente all'interno del file .dia nella cartella doc/).~~ <br>
2. Finire la pianificazione delle AZIONI. <br>
    1. Leggere documentazione di QStackedWidget per capire come aggiungere i vari Widget. <br>
    2. definire i vari widget da poter aggiungere alla stack di widget e dove inserire (in quale parte della mainWindow) la 
        stack di widget. <br>

### MODEL
1. aggiungere ad AbstactProduct.h il campo std::string imagePath
2. aggiungere ad AbstactProduct.h il campo std::string description 
3. Rinominare le interfacce appartenenti alla persistenza dei dati in quanto non sono proprio delle interfacce pure. <br>
    In particolare queste sono delle classi astratte (ex: IFile -> AbstractFile, IReader -> AbstractReader).

### GUI 
~~1. Guardare il file creato in GoodNotes e cercare la documentazione degli elementi della gerarchia segnati in rosa.~~
2. ~~cercare la documentazione della scrollArea all'interno della documentazione di Qt.~~
3. ~~Studiare come il search engine di Zanella implementa la persistenza dei dati in JSON.~~
4. Risolvere il problema di renderizzare vari itemRenderer utilizzando le classi aggiunte nel commit #4.
5. Studiare come il search engine di Zanella implementa la visualizzazione dei dati. 
6. Pianificare la GUI per passi: ogni passo corrisponde all'implementazione di un'azione che si vuole ottenere.

### AZIONI

#### CREAZIONE FILE DATASET 
(la soluzione sta semplicemente nel vedere l'implementazione del boy zanella) <br>
CHI LA COMPIE: bottone => signal => slot (bottone della toolbar presente nella mainwindow) <br>
DOVE: la funzione che crea il dataset deve essere uno slot della mainWindow <br>
OPZIONALE: questo slot possiamo far si che richiami la funzione di importazione del dataset, cosi' che in automatico la creazione del dataset <br>
ne comporti subito l'importazione.
COSA FA: la funzione crea un nuovo file all'interno della cartella selezionata. Poi esegue questi passi (in ordine):

1. richiama la funzione di importazione del dataset.
2. siccome e' nella mainWindow => ha accesso ai suoi campi privati => mediante i puntatori ai widget richiama i metodi per fornire controllo ad essi, e quindi abilitarli.

#### IMPORTAZIONE DATASET 
Quest'azione corrisponde alla serializzazione degli oggetti (scrittura, da oggetti ad un formato scelto, nel file di dataset creato) <br>
Nella soluzione di Zanella questo risulta abbastanza chiaro come farlo. in ogni caso per farlo bisogna usare le classi che ho implementato per serializzare/deserializzare i dati <br>

CHI LA COMPIE: bottone => signal => slot (bottone della toolbar presente nella mainwindow) <br>
DOVE: la funzione slot nella main window chiamata loadFromDataset(); <br>
COSA FA: la funzione serializza i dati in questo modo:

1. Tramite il puntatore a file, legge il contenuto del file => ritorna un std::vector<AbstractProduct*>. <br>
2. Inserire nel buffer tutti i prodotti. questo torna poi utile per molti tipi di controlli che possono essere fatti sull'inserimento di un nuovo prodotto. 

Il codice dovrebbe essere una cosa di questo tipo:
```cpp
void loadDataset() {
    JsonReader reader()
    JsonConverter converter(reader);
    std::vector<AbstractProduct*> aux = fileHandle->ReadFrom(converter);

    for(auto it = aux.begin(); it != aux.end(); it++) {
        buffer.insert((*aux).getId(), aux);
        contenitore->insert(aux);
    }
}
```

#### ESPORTAZIONE DATASET (importazione degli oggetti in memoria da uno dei possibili file creati in precedenza).

CHI LA COMPIE: bottone => signal => slot (bottone della toolbar presente nella mainwindow) writeToFile(). <br>
DOVE: la funzione slot nella main window chiamata `writeToFile();` <br>
COSA FA: tramite puntatore al file `jsonFile JsonFile*`  nella MainWindow, si utilizza il metodo WriteTo(const vector<AbstractProduct*>&, const JsonConverter&); <br>
All'interno del signal `writeToFile()`: 

1. Prendere il contenuto del buffer (tramite il puntatore presente nella MainWindow) e mettere tutti i valori all'interno di un <br>
`std::vector<AbstractProduct*> x`.
2. Passare `x` alla funzione JsonFile& WriteTo. Questa garantisce di scrivere all'interno del file che si trova al percorso <br>
specificato dall'oggetto puntato dal puntatore AbstractFile* presente nella MainWindow. <br>
NOTA BENE: Questa funzione lascia intatto il buffer.

```cpp
void writeDataset() {
    std::vector<AbstractProduct*> aux();

    for(std::vector<AbstractProduct*>::iterator it = aux.begin(); 
        it != aux.end(); 
        it++) {
        aux.pushBack(buffer[(*it)->getId()]);
    }
    
    JsonReader reader();
    JsonConverter converter(reader);
    fileHandle->writeToFile(aux, converter);

    std::cout << "Dataset Salvato" << std::endl;
    unsavedChanges = false;
}
```

ATTENZIONE: nella funzione writeToFile viene creato un QJsonArray; questo contiene dei QJsonObject. <br>
I `QJsonObject` vengono creati diversamente in base al tipo dinamico del puntatore ad `AbstractProduct*`, il che suggerisce che si debba usare il visitor pattern
per generare diversi QJsonObject. <br>

Questo lo si risolve creando la seguente classe, che implementa l'interfaccia IConstVisitor. <br>
Viene implementata questa in quanto i puntatori agli oggetti sono soplamente utili in lettura. 

```cpp
#include <QJsonObject>
#include "IConstProductVisitor.h"

class JsonVisitor: public IConstProductVisitor {
private:
    QJsonObject jsonObject;
public:
    virtual void visit(const Virtuale*);
    virtual void visit(const Fisico*);
    virtual void visit(const Noleggio*);
    QJsonObject getJsonObject();
};

```

#### CREAZIONE DI UN PRODOTTO DA AGGIUNGERE AL CATALOGO.

CHI LA COMPIE: bottone presente all'interno della toolbar nella mainWindow => signal clicked() => slot. <br>
DOVE: lo slot è all'interno della MainWindow chiamato `createItem()`. <br>
COSA FA: la funzione slot esegue questo. <br>

1. Pulisce la stack di widget all'interno della MainWindow tramite il puntatore `QStackedWidget* stackedWidget` per poi far comparire un widget chiamato `ItemCreator`. <br>
La soluzione di Zanella riadattata al mio modello della GUI è il seguente:

```cpp
void MainWindow::createItem() {
    clearStack(); // da implementare
    QScrollArea* scrollArea = new QScrollArea(); 
    scroll_area->setHorizontalScrollBarPolicy(Qt::ScrollBarAsNeeded);
    scroll_area->setVerticalScrollBarPolicy(Qt::ScrollBarAsNeeded);
    scroll_area->setWidgetResizable(true);
    ItemCreator* createWidget = new ItemCreator(nullptr, this);
    scroll_area->setWidget(createWidget);
    stacked_widget->addWidget(scrollArea);
    stacked_widget->setCurrentIndex(1);
    has_unsaved_changes = true;
    showStatus("Creating a new item.");
}
```

2. All'interno dello `SLOT(apply())`, triggerato dal `QPushButton* buttonCreate` all'interno dell' `ItemCreator` controllo che l'identificatore dell' `AbstractProduct*` non sia già presente all'interno del buffer. Siccome il puntatore al buffer è contenuto nella mainWindow, `ItemCreator` deve avere un puntatore alla mainWindow. 

Il buffer internamente è una map\<unsigned int, AbstractProduct\*\>; => la verifica di ciò può essere fatta in questo modo: 

`if (buffer[(\*AbstractProduct).getId()] == null) then OK, else NOT OK.`

4. se OK => il prodotto appena creato può essere inserito all'interno del catalogo. In particolare deve essere inserito all'interno del buffer, all'interno del contenitore di AbstractProduct* e deve essere creato un listItem all'interno della scrollArea per poter rappresentare l'oggetto appena creato.
5. se NOT OK => il prodotto non puà essere inserito. Di conseguenza quello che si può fare è far apparire una finestra di dialogo che dice che il seguente prodotto non può essere inserito.

Il codice è più o meno questo.

```cpp
void ItemCreator::apply() {
    Item::AbstractProduct* item = editor->create();
    // setup oggetti che servono per i controlli.
    Buffer* buffer = mainWindow->getBuffer();
    const std::map<unsigned int, AbstractProduct*>& m = buffer->getMap();

    if (m[item.getId()) {
        std::cout << "elemento non inserito in quanto l'identificatore esiste già" << std::endl;
        delete item; // questo in quanto, nel caso sia gia' presente un item avente lo stesso id
                     // non serve piu' questo item in memoria.
    } else {
        (*buffer).insert(item);
        mainWindow->reloadData(); // dubbio 
        mainWindow->getSearchWidget()->search(); // dubbio
    }
}
```
NOTA BENE: secondo la struttura riadattata degli oggetti grafici per la creazione e modifica del prodotto, nella classe `AbstractEditor` deve esserci un metodo create, in quanto ogni editor deve poter creare un AbstractProduct* avente tipo dinamico diverso. <br>
Il metodo create di un determinato prodotto implementa la logica seguente; prendiamo, per esempio la creazione di un prodotto fisico a noleggio: <br>

```cpp
AbstractProduct* EditorNoleggio::create() {
    // prendere i valori dagli elementi grafici
    int identifier = boxId->value();
    QString name = boxName->text();
    QString prezzo = boxPrezzo->value();
    QString imagePath = boxImagePath->text();
    QString noleggiatore = boxNoleggiatore->text();
    QString noleggiante = boxNoleggiante->text();

    // passare alla creazione di AbstractProduct*
    Item::AbstractProduct* item = new Noleggio(identifier, name, description, imagePath, noleggiatore, noleggiante);
}
```

#### VISUALIZZAZIONE DI UN PRODOTTO AGGIUNTO AL CATALOGO.
Questo in realtà è abbastanza opzionale in quanto una visualizzazione (lista di ListItem nella scrollbar) la si ha già.  

#### MODIFICA DI UN PRODOTTO AGGIUNTO AL CATALOGO.

CHI LA COMPIE: bottone presente all'interno del ListItem => signal clicked => slot. <br>
DOVE: lo slot è all'interno della mainWindow. <br>
PROBLEMI: praticamente sono le stesse problematiche dell'eliminazione di un prodotto in termini di segnali e slot con qualche complicazione aggiuntiva. <br>
Per vedere la propagazione di `AbstractProduct*` alla mainwindow, andare alla sezione di [eliminazione](#eliminazione-di-un-prodotto-aggiunto-al-catalogo)

1. La modifica dipende dal tipo del prodotto che si vuole modificare.
2. L'editor di un prodotto deve poter modificare il prodotto "giusto", nel senso che deve modificare il prodotto puntato dal listItem che ha segnalato <br> 
il click del pulsante edit.
3. dopo la modifica del prodotto, bisogna renderla visibile all'interno del resultsWidget.

SOLUZIONE 1: Dato che ogni listItem ha un bottone modifica prodotto => signal. <br>
Questo signal deve essere connesso ad uno slot presente nella mainWindow chiamato `updateItem(AbstractProduct*)`.
Per la propagazione del segnale vedere la soluzione descritta nel problema dell'eliminazione. <br>

Siamo ora all'interno della funzione `updateItem(AbstractProduct*)`:

```cpp
void MainWindow::updateItem(const AbstractProduct* item) {
    stackedWidget->clearstack(); // copiare questo metodo da Zanella. 
}
```
In base al tipo dinamico di item, si puo' renderizzare e mettere all'interno dello stackedWidget uno degli editor per ognuno dei tipi (EditorFisico, EditorVirtuale e EditorNoleggio). <br>
Questo lo si può fare tramite il Visitor design pattern (in particolare sfruttiamo la versione IConstVisitor di questo).

```cpp
class Virtuale;
class Fisico;
class Noleggio;

class IConstProductVisitor {
public:
    virtual ~IConstProductVisitor() {};
    virtual void visit(const Virtuale*) = 0;
    virtual void visit(const Fisico*) = 0;
    virtual void visit(const Noleggio*) = 0;
};
```
Quindi possiamo creare una classe ItemEditorRender che implementa quest'interfaccia. la classe è fatta in questo modo:

```cpp
#include "IConstVisitor.h"

class ItemEditorRenderer : public IConstVisitor {
private:
	QWidget* widget;
public:
	virtual void visit(const Virtuale*);
	virtual void visit(const Fisico*);
	virtual void visit(const Noleggio*);	
	QWidget* getRenderedWidget() const;
};
```
Ognuno di questi metodi ha un implementazione diversa, analoga al seguente esempio: 

```cpp
virtual void visit(const Fisico* element) {
    // render EditorFisico* di tipo QWidget.
    EditorFisico* editor = new EditorFisico(nullptr, element);
    widget = editor;
}
```
Di conseguenza, per ognuno dei tipi, si devono implementare degli editor appositi: EditorFisico, EditorVirtuale e EditorNoleggio. <br>
A questo punto, il signal updateItem() diventa così.

```cpp
void MainWindow::updateItem(AbstractProduct* item) {
    stackedWidget->clearstack(); // copiare questo metodo da Zanella. 
    IConstVisitor* editorRenderer = new ItemEditorRenderer(item);
    item.accept(editorRenderer);
    QWidget* editor = editorRenderer.getRenderedWidget();
    stackedWidget->addWidget(editor);
}
```

SOLUZIONE 2
Sappiamo che ogni listItem ha un puntatore al prodotto che vuole rappresentare; <br>
Questo viene propagto al signal editItem(AbstractProduct\*) presente nella mainWindow.
La soluzione in realtà è nella struttura dell'AbstractEditor, in quanto, quando viene invocato il metodo visit, si crea un editor ad hoc passandogli il puntatore 
al prodotto da modificare. <br>
Una volta modificato questo prodotto, si può propagare il cambiamento alla MainWindow. <br>
La struttura della classe Editor è la seguente:

```cpp
class AbstractEditor : public QWidget {
private:
	AbstractProduct* subject;
    QLineEdit* boxId;
    QSpinBox* boxPrezzo;
    QLineEdit* boxNamel;
    QLineEdit* boxImagePath;
public:
	virtual ~AbstractEditor() {};
	AbstractEditor(AbstractProduct* item, QWidget* parent = nullptr) : QWidget(parent), subject(item) {} // costruttore di default.
			
public slots:
    void update(); // da collegare al bottone apply changes 
    void create(); // da collegare al bottone apply
signals:
	// SIGNALS
};
```

SOLUZIONE 3

Per propagare la modifica dell'item si possono adottare diverse strategie. la più semplice nel mio caso potrebbe essere la seguente. 
Modificare la classe AbstractEditor in questo modo.

```cpp
class AbstractEditor : public QWidget {
private:
	AbstractProduct* subject;
	MainWindow* mainWindow; // AGGIUNTO
    QLineEdit* boxId;
    QSpinBox* boxPrezzo;
    QLineEdit* boxNamel;
    QLineEdit* boxImagePath;   
    QPushButton* applyChangesButton; // AGGIUNTO
public:
	virtual ~AbstractEditor() {};
	AbstractEditor(AbstractProduct* item, QWidget* parent = nullptr) : QWidget(parent), subject(item) {} // costruttore di default.
    void setMainWindow(MainWindow* w); // AGGIUNTO
public slots:
    void update();  
    void create(); 
};
```

Il metodo setMainWindow(MainWindow* w) è definito così:

```cpp
void setMainWindow(MainWindow* w) {
    mainWindow = w;    
};
```

All'interno del metodo update posso fare tutto quello che c'è da fare in quanto ho il puntatore alla mainWindow:
1. Controllare se l'update che si vuole applicare sia fattibile oppure no; in caso lo fosse:
    1. Richiamare l'update dell'item all'interno dela mainWindow tramite l'api della memoria.
    2. Richiamare l'update del buffer tramite la sua API.
    3. fare il refresh degli elementi grafici (*)

REFRESH ELEMENTI GRAFICI
All'interno del ResultsWidget, ho un `std::vector<ListItem*>`.  <br>
Dato che, nella MainWindow vi è un puntatore al fantastico oggetto resultsWidget, quello che posso fare è creare <br>
un metodo refresh(), da porre all'interno della MainWindow.

Questo metodo scorre il buffer di elementi e, per ogni elemento (\*) aggiorna il contenuto. <br>
Siccome `std::vector<ListItem*>` può contenere dei ListItem Diversi, allora ogni ListItem deve poter implementare la sua logica di refresh. 
Questo tuttavia risulta problematico in quanto la rappresentazione di ListItem ha solamente un puntatore ad un QWidget generato al momento del rendering.

SOLUZIONE 3.1
è veramente necessario fare il refresh di ogni elemento grafico? d'altra parte, se modifico un prodotto, modifico solo quel prodotto. <br>
è naturale domandarsi come possa modificare solamente l'oggetto grafico che lo rappresenta.

#### ELIMINAZIONE DI UN PRODOTTO AGGIUNTO AL CATALOGO
CHI LA COMPIE: bottone presente all'interno del ListItem => signal clicked => slot. <br>

PROBLEMI:
1. All'interno della MainWindow: ho solamente i puntatori ai seguenti widget: SearchWidget e FilterWidget. <br>
2. Lo slot `deleteItem(AbstractProduct*)` all'interno della MainWindow deve essere connesso ad un segnle affinché possa essere eseguito. <br>

PRIMA SOLUZIONE
1. ListItem connette il segnale clicked() di QPushButton ad uno slot (privato in quanto serve solamente a questo widget per poter emettere il segnale) che emette un segnale del tipo `signal(AbstractProduct*)`.
2. All'interno del SearchWidget aggiungere il medesimo slot privato che fa esattamente la stessa cosa. 
3. Collegare il segnale emesso dal SearchWidget allo slot deleteItem() tramite il puntatore SearchWidget presente nella MainWindow. <br>

Il segnale che viene emesso sia da listItem che dallo slot può chiamarsi `itemDeleted(AbstractProduct*)`. <br>
Questo slot privato deve essere connesso al tasto eslimina durante la costruzione di `ListItem`. <br>
Essa avviene circa in questo modo:

```cpp
ListItem::ListItem(QWidget* parent) : QWidget(parent) {
    //... inizializzazine oggetti grafici per rappresentare un AbstractProduct*

    connect(editButton, SIGNAL(clicked()), this, SLOT(slotItemSelected()));         
    connect(deleteButton, SIGNAL(clicked()), this, SLOT(slotItemDeleted()));         
}
```
Lo slot `slotItemDeleted()` all'interno di `ListItem` implementa la seguente logica:

```cpp
ListItem::slotItemSelected() {
    emit(item);
}

```
Per quanto riguarda la connessione con il resultWidget, questa viene fatta nel momento della costruzione

```cpp
ResultsWidget::ResultsWidget(QWidget* parent) : QWidget(parent) {
    
    // dal puntatore alla mainwindow prendiamo il contenitore, nel quale dobbiamo inserire un metodo getData() che ritorna un std::vector<AbstractProduct*>
    std::vector<AbstractProduct*> vettore = (mainWindow->getContenitore())->getData();

    // inizializzazione std::vector<ListItem*> 
    for(auto it = vettore.begin(); it != vettore.end(); it++) {
        ListItem* listItem = new ListItem(this, *it);        
        connect(listItem, SIGNAL(signalDeltedItem(AbstractProduct*), mainWindow, SLOT(deleteItem(AbstractProduct*)));
    }
}
```

Questa logica si può riutilizzare per la propagazione di `AbstractProduct*` durante la'operazione di modifica, la quale scatta tramite il `SIGNAL(clicked())` di editButton all'interno di `ListItem`.

SECONDA SOLUZIONE (scartata)

Si basa sull'utilizzo del campo parent del widget ListItem.
1. Ogni qual volta che inizializzo un ListItem e lo aggiungo alla lista presente all'interno del searchWidget, inizializzo <br>
il suo campo parent con il puntatore al searchWidget.
2. Creo un slot privato all'interno del widget chiamato callForDeleteItem() (il nome è solo un esempio). Questo slot emette un segnale del tipo signal( AbstractProduct* ).


# IDEE

1. Mio ResultWidget: utilizzare una QScrollArea per poter mostrare i risultati di un filtro 
2. Classe filter: modella un filtro che viene fatto sugli oggetti del contenitore. 
3. Avere tre bottoni diversi per creare un articolo da inserire nel catalogo (fisico, virtuale, noleggio). Quando uno di <br> questi viene clicckato, emette un segnale, il quale triggera l'editor giusto per poter creare un nuovo prodotto <br>
da inserire nel catalogo.
4. Utilizzare la classe StackedWidget per poter disporre al posto del resultWidget i vari itemEditor. 
5. Provare a vedere se eliminando degli elementi programmaticamente dalla QScrollarea questi mantengano una disposizione "senza buchi".  


### LINK UTILI

https://doc.qt.io/qt-6/qstackedwidget.html#count-prop

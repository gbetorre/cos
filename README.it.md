### 2 lingue disponibili
[![en](https://img.shields.io/badge/lang-en-red.svg)](https://github.com/gbetorre/cos/blob/master/README.md)
[![it](https://img.shields.io/badge/lang-it-yellow.svg)](https://github.com/gbetorre/cos/blob/master/README.it.md)

---

![Servlet license](https://img.shields.io/badge/license-Servlet-blue)

Questo repository &egrave; il porting di una classica utility scritta originariamente in Java da [Jason Hunter.](https://www.servlets.com/jason/) 

# Storia

Questa libreria è un'utility che offre diversi strumenti utili per agevolare lo sviluppo di Servlet e classi affini.

E' stata originariamente rilasciata da Jason Hunter (jhunter_AT_servlets.com); successivamente, è stata portata su GitHub da [James Zhan](https://github.com/jfinal) ([jfinal](https://github.com/jfinal)), rendendola anche facilmente integrabile nei progetti Maven tramite la seguente dipendenza:

```XML
    <dependency>
        <groupId>com.jfinal</groupId>
        <artifactId>cos</artifactId>
        <version>2017.5</version>
    </dependency
```

La [release di jfinal](https://github.com/jfinal/cos) è molto utile per gli sviluppatori Java che sono abituati a utilizzare le classi di questa libreria ma non vogliono farlo integrando nel loro progetto direttamente il jar (scaricabile da https://www.servlets.com/cos/) e piuttosto preferiscono configurare la libreria nel descrittore del progetto Maven.

# Il problema

Tutto è andato bene fino alla versione 9 di Tomcat.

Consideriamo che stiamo parlando di un'utility destinata ad essere integrata in applicazioni di tipo web-based, che girano dentro un server, ricevono richieste e forniscono risposte sostanzialmente attraverso il protocollo HTTP.
Uno dei server più utilizzati a questo scopo, soprattutto in fase di sviluppo, è Apache Tomcat, giunto attualmente alla versione 11.

Il problema è che, come riportato anche nella documentazione, a partire dalla versione 10:

> come risultato del trasferimento da Java EE a Jakarta EE, a sua volta dipendente dal trasferimento di Java Enterprise Edition all'Eclipse Foundation, il package di base per tutte le API implementate è cambiato da javax.* a jakarta.*.
> Questo quasi certamente (ndr.: anche senza "quasi") richiederà modifiche del codice per poter migrare le applicazioni da Tomcat 9 - e precedenti - a Tomcat 10 e successivi.
> Un tool di migrazione è disponibile per aiutare in questo processo.

Il che significa che, se non si effettua la migrazione, le applicazioni scritte e funzionanti sotto Tomcat 9, in Tomcat 10 e successivi non funzioneranno più.

Riassumendo:
- Da **Tomcat 10** in poi, il package delle API Servlet è cambiato da **`javax.servlet.*` a `jakarta.servlet.*`** a causa della migrazione del **namespace Jakarta EE**.

- **Servlets e relative classi, come `HttpServlet`, ora sono accessibili da `jakarta.servlet.http.HttpServlet`**, non più da `javax.servlet.http.HttpServlet`.
    
- Le Web apps compilate con il vecchio import `javax.servlet` utilizzato nei sorgenti daranno una `ClassNotFoundException` su Tomcat 10, e successivi, a meno di migrare il package degli import al nuovo namespace `jakarta.servlet`.

# La soluzione

Al fine di rendere compatibile la libreria `cos` con il nuovo namespace di Jakarta EE, bisogna effettuare sui sorgenti le operazioni seguenti:

1. **Editare tutti i files sorgenti:**

- Cambiare tutti gli import da:<br> 
    `import javax.servlet.http.HttpServlet;`<br> 
    a:<br> 
    `import jakarta.servlet.http.HttpServlet;`

- Implementare i metodi astratti ereditati (anche in forma di stub) ove necessario.

2. Per poter compilare il progetto per Jakarta EE, naturalmente bisogna anche **far riferimento ai namespace Jakarta EE 9+ nella configurazione di Maven:**

```XML    
    <dependency> 
        <groupId>jakarta.servlet</groupId> 
        <artifactId>jakarta.servlet-api</artifactId> 
        <version>5.0.0</version> 
        <scope>provided</scope>
    </dependency>
```

Lo scopo di questo repository è di fare per voi queste operazioni, rendendo il lavoro del programmatore che ama utilizzare la libreria `cos` più semplice e risparmiargli un po' di lavoro (visto che io l'ho già fatto per i miei progetti).

Se questo intento ti è servito, o ti è piaciuto, mi farebbe piacere se mettessi una stella al progetto!

# Uso

Quando la release verrà rilasciata, per far riferimento alla libreria basterà aggiungere, ad esempio:

```XML
    <dependency>
        <groupId>com.gbetorre</groupId>
        <artifactId>cos</artifactId>
        <version>2025.10</version>
    </dependency
```

# Esempi

Una funzionalità di `cos` che trovo molto utile è, ad esempio, la gestione dei parametri della HttpServletRequest da parte del `com.oreilly.servlet.ParameterParser`:

```JAVA
import com.oreilly.servlet.ParameterParser;
public class ProcessCommand extends ItemBean implements Command {
    public void execute(HttpServletRequest req) {
        ParameterParser parser = new ParameterParser(req);
        String part = parser.getStringParameter("p", "-");
        int idP = parser.getIntParameter("pliv", -1);
        String pliv = parser.getStringParameter("pliv2", "");
        int liv = parser.getIntParameter("liv", -1);
    }
}
```
***Listato.1 - Esempio di utilizzo di una classe del cos, ParameterParser***

Come si vede, usando il ParameterParser al posto del recupero diretto da HttpServletRequest, diventa possibile assegnare un default al valore, in caso non sia presente nella richiesta. Ciò sembra banale ma invece è molto comodo perché risparmia di dover assegnare i default in più istruzioni, gestire le eccezioni derivanti dalla presenza di valori null, etc.

Un altro esempio dell'uso sempre di questa classe può essere visto nel contesto del popolamento di strutture afferenti al tipo Dictionary:

```JAVA
LinkedHashMap<String, String> struct = new LinkedHashMap<>();
struct.put("liv1",  parser.getStringParameter("sliv1", VOID_STRING));
struct.put("liv2",  parser.getStringParameter("sliv2", VOID_STRING));
struct.put("liv3",  parser.getStringParameter("sliv3", VOID_STRING));
struct.put("liv4",  parser.getStringParameter("sliv4", VOID_STRING));
```
***Listato.2 - Esempio di utilizzo di ParameterParser in HashMap***

## Nota stilistica

Se si esaminano i sorgenti del repository cos, si noterà che la sintassi adottata in queste classi è ormai un po' obsoleta, rispetto agli standard moderni.

Ad esempio, 
* vengono usati alcuni costruttori deprecati (p.es. Boolean(String), Double(String), Float(String)...)
* vengono usati alcuni oggetti con implementazioni sorpassate (p.es. Hashtable invece di HashMap)
* vengono usate alcune soluzioni che oggi vengono considerate obsolete (ad esempio accedere ad elementi di una collezione in modalità raw piuttosto che utilizzando i generics)

E' normale che le potenzialità della moderna sintassi di Java non venga sfruttata, dal momento che, quando queste classi sono state scritte, queste funzionalità ancora non esistevano nel linguaggio. Inoltre, all'epoca molti metodi e costruttori non erano stati ancora deprecati.

Non c'è quindi qui alcuna intenzione di criticare il lavoro di scrittura fatto a suo tempo dall'autore.
Semplicemente, bisogna tener conto che alcune scelte sono da inquadrare nel periodo storico nelle quali sono state intraprese.
Inoltre, personalmente non ho intenzione di alterare il lavoro originale più di tanto, limitandomi semplicemente a rendere la libreria compatibile con le moderne API di Servlet ma non riformulandone lo stile di scrittura.

## Nota sul progetto

Lo scopo di questo porting è infatti semplicemente quello di rendere disponibile per l'uso questa libreria dopo il passaggio dell'implementazione di Tomcat a seguito del trasferimento di Java EE all'Eclipse Foundation.

Certo, Apache Tomcat mette a disposizione un tool di migrazione per questo scopo, ma avere una libreria già pronta non dev'essere poi così male...

`cos` è un tool utilizzato da una nicchia di sviluppatori e questo suo porting serve per continuare a tenere il codice dei progetti funzionante senza dover perdere tempo anche ad effettuare gli adeguamenti: gli sviluppatori senior sanno bene quanto tempo e quanto stress comportano le migrazioni e i nuovi startup di progetto.

Spero pertanto che questo porting da Java EE a Jakarta EE risulti utile per alcuni di noi programmatori e, in tal caso, apprezzerei che venisse messa una stella a questo progetto.

# Licenza

Servlet, v. LICENSE.

# Autori

* [jhunter](https://www.servlets.com/jason/) 

## Revisori

* [jfinal](https://github.com/jfinal) 
* [gbetorre](https://github.com/gbetorre)





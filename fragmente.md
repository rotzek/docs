# Fragmente

Fragmente werden in REDAXO eingesetzt, um wiederkehrende Codeschnippsel übersichtlich zu verwalten. Fragmente sind PHP Dateien und werden bei der Ausgabe geparst, sodass Variablen ausgegeben und verarbeitet werden können. In REDAXO selbst werden zahlreiche Fragmente für die Ausgabe des Backends verwendet. Diese Fragmente können auch als Ausgangsbasis für eigene Fragmente genutzt werden. Grundsätzlich wird ein Fragment in dieser Form angesrochen und ausgegeben:

    $fragment = new rex_fragment();
    $fragment->parse('meinfragment.php');
    
Fragmente, die in addons im Verzeichnis fragments abgelegt werden, werden ohne Pfadangabe gefunden.

## Variablen in Fragmenten

Fragmente können Variablen ausgeben, die zuvor dem Fragment per setVar zugewiesen wurden.

    $fragment->setVar('meinevar','Ich bin der Inhalt',false);
    
Die Ausgabe im Fragment erfolgt per

    echo $this->meinevar;
    
oder

    echo $this->getVar('meinevar');
    
Wenn eine Variable vom Typ Array übergeben wird, so kann dieses Array durchlaufen werden:

    $fragment->setVar('monate',['Januar','Februar','März','April','Mai','Juni','Juli','August','September','...']);
    
Ausgabe im Fragment:

    <ul>
    <?php foreach ($this->monate as $monat) : ?>
      <li><?= $monat ?></li>
    <?php endforeach ?>
    </ul>

    
## REDAXO Fragmente nutzen

Die in REDAXO vorliegenden Fragmente können für die eigene Ausgabe im Frontend oder im Backend genutzt werden.

### Beispiel Paginierung

Bei der Ausgabe von Datensätzen oder längeren Listen wird häufig eine Paginierung verwendet. Eine Paginierung steht in REDAXO bereits zur Verfügung. Diese Paginierung kann für eigene Zwecke verwendet und angepasst werden.

#### Modulausgabe für die Paginierung
    
    // Anzahl Datensätze pro Seite
    $rows_per_page = 15;
    
    if ("REX_VALUE[1]") {
       $sql = rex_sql::factory();
       
       // Select für alle anzuzeigenden Datensätze
       $qry = "SELECT name, shorttext, id FROM rex_meinetabelle";
       $sql->setQuery($qry);
       
       // Anzahl Zeilen insgesamt (
       $rows = $sql->getRows();
       
       // Query erweitern um Anzahl der Datensätze pro Seite
       $qry .= ' LIMIT '.$rows_per_page;
       
       // Wenn nicht die erste Seite angezeigt wird, Offset an Query anhängen
       if (rex_get('start','int',0)) {
          $qry .= ' OFFSET '.rex_get('start','int',0);
       }
       
       // $res wird später für die Anzeige der Liste durchlaufen
       $sql->setQuery($qry);
       $res = $sql->getArray();   
    } 
    
    // Neues Paginierungsobjekt erstellen
    $pager = new rex_pager($rows_per_page);
    
    // Gesamtanzahl der Zeilen zuweisen
    $pager->setRowCount($rows);

    // Neues Fragment Objekt erstellen
    $fragment = new rex_fragment();
    
    // pager Objekt an Fragment übergeben
    $fragment->setVar('pager', $pager, false);
    
    // urlprovider definieren
    $fragment->setVar('urlprovider', rex_article::getCurrent());

    // Paginierung ausgeben
    echo $fragment->parse('core/navigations/pagination.php');

Hierbei wird das Fragment /core/fragments/core/navigations/pagination.php für die Ausgabe verwendet. Dieses Fragment kann auch in ein eigenes Addon in das Verzeichnis fragments kopiert und geändert werden. Wenn es dort unter dem Namen mypagination.php abgelegt wird, so kann es ohne Pfadangabe aufgerufen werden:

    echo $fragment->parse('mypagination.php');


#### Beispiel 2 für die Paginierung

$pager = new rex_pager(10);
$table = rex_yform_manager_table::get('rex_meinetabelle');
$ergebnisse = $table->query()
    ->paginate($pager);
$fragment = new rex_fragment();
$fragment->setVar('urlprovider', rex_article::getCurrent());
$fragment->setVar('pager', $pager);
echo $fragment->parse('core/navigations/pagination.php');

foreach ($ergebnisse as $erg) {
echo "ID: ".$erg->id;
}
echo $pager->getRowCount();
echo $pager->getCurrentPage();
echo $pager->getLastPage();
echo $pager->getPageCount();

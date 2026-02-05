
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <title>Gestion des tâches</title>
    <link rel="stylesheet" href="Mini_projet.css">
</head>
<?php
$fichier = "taches.json";


if (!file_exists($fichier)) {
    file_put_contents($fichier, json_encode([]));
}

$taches = json_decode(file_get_contents($fichier), true);
if (!is_array($taches)) {
    $taches = [];
}


if (isset($_POST['ajouter'])) {
    $taches[] = [
        "id" => time(),
        "titre" => $_POST['titre'],
        "description" => $_POST['description'],
        "priorite" => $_POST['priorite'],
        "statut" => "à faire",
        "date_creation" => date("Y-m-d"),
        "date_limite" => $_POST['date_limite']
    ];
    file_put_contents($fichier, json_encode($taches, JSON_PRETTY_PRINT));
    header("Location: Mini_projet.php");
    exit();
}

/* ===== CHANGER STATUT ===== */
if (isset($_GET['statut'])) {
    foreach ($taches as &$tache) {
        if ($tache['id'] == $_GET['statut']) {
            if ($tache['statut'] == "à faire") {
                $tache['statut'] = "en cours";
            } elseif ($tache['statut'] == "en cours") {
                $tache['statut'] = "terminée";
            }
        }
    }
    unset($tache);
    file_put_contents($fichier, json_encode($taches, JSON_PRETTY_PRINT));
    header("Location: Mini_projet.php");
    exit();
}

/* ===== SUPPRESSION ===== */
if (isset($_GET['supprimer'])) {
    $nouveau = [];
    foreach ($taches as $tache) {
        if ($tache['id'] != $_GET['supprimer']) {
            $nouveau[] = $tache;
        }
    }
    $taches = array_values($nouveau);
    file_put_contents($fichier, json_encode($taches, JSON_PRETTY_PRINT));
    header("Location: Mini_projet.php");
    exit();
}


$recherche = $_GET['recherche'] ?? "";
$filtreStatut = $_GET['filtreStatut'] ?? "";
$filtrePriorite = $_GET['filtrePriorite'] ?? "";

$tachesAffichees = [];
foreach ($taches as $tache) {
    if (
        (stripos($tache['titre'], $recherche) !== false ||
        stripos($tache['description'], $recherche) !== false) &&
        ($filtreStatut == "" || $tache['statut'] == $filtreStatut) &&
        ($filtrePriorite == "" || $tache['priorite'] == $filtrePriorite)
    ) {
        $tachesAffichees[] = $tache;
    }
}

$total = count($taches);
$terminees = 0;
$retard = 0;

foreach ($taches as $tache) {
    if ($tache['statut'] == "terminée") {
        $terminees++;
    }
    if ($tache['statut'] != "terminée" && $tache['date_limite'] < date("Y-m-d")) {
        $retard++;
    }
}

$pourcentage = ($total > 0) ? round(($terminees / $total) * 100) : 0;
?>

<body>

<h1> GESTION DES TACHES </h1>

<form method="post" class="form">
    <input type="text" name="titre" placeholder="Titre" required>
    <textarea name="description" placeholder="Description" required></textarea>
    <select name="priorite">
        <option value="basse">Basse</option>
        <option value="moyenne">Moyenne</option>
        <option value="haute">Haute</option>
    </select>
    <input type="date" name="date_limite" required>
    <button type="submit" name="ajouter">Ajouter</button>
</form>

<form method="get" class="form">
    <input type="text" name="recherche" placeholder="Recherche...">
    <select name="filtreStatut">
        <option value="">Tous les statuts</option>
        <option value="à faire">À faire</option>
        <option value="en cours">En cours</option>
        <option value="terminée">Terminée</option>
    </select>
    <select name="filtrePriorite">
        <option value="">Toutes priorités</option>
        <option value="basse">Basse</option>
        <option value="moyenne">Moyenne</option>
        <option value="haute">Haute</option>
    </select>
    <button type="submit">Filtrer</button>
</form>

<table>
    <tr>
        <th>Titre</th>
        <th>Priorité</th>
        <th>Statut</th>
        <th>Date limite</th>
        <th>Actions</th>
    </tr>

<?php foreach ($tachesAffichees as $tache): 
    $enRetard = ($tache['statut'] != "terminée" && $tache['date_limite'] < date("Y-m-d"));
?>
    <tr class="<?php echo $enRetard ? 'retard' : ''; ?>">
        <td><?php echo $tache['titre']; ?></td>
        <td><?php echo $tache['priorite']; ?></td>
        <td><?php echo $tache['statut']; ?></td>
        <td><?php echo $tache['date_limite']; ?></td>
        <td>
            <a href="?statut=<?php echo $tache['id']; ?>">Changer</a> |
            <a href="?supprimer=<?php echo $tache['id']; ?>" onclick="return confirm('Supprimer cette tâche ?')">Supprimer</a>
        </td>
    </tr>
<?php endforeach; ?>
</table>

<div class="stats">
    <p>Total : <?php echo $total; ?></p>
    <p>Terminées : <?php echo $terminees; ?></p>
    <p>Pourcentage : <?php echo $pourcentage; ?>%</p>
    <p>En retard : <?php echo $retard; ?></p>
</div>

</body>
</html>


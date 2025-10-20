# Μικρομάθημα 6: Πρόσβαση σε Components δικού μας GameObject και άλλων GameObjects (Unity)

**Μάθημα/Ενότητα:** Interactive Systems Prototyping — Unity Microlessons  
**Στόχος:** Να μάθουμε πρακτικούς τρόπους πρόσβασης στα components ενός GameObject, καθώς και σε components άλλων GameObjects, και να τα χειριζόμαστε (π.χ. αλλαγή χρώματος, εμφάνιση/κρύψιμο renderer, προσθήκη βαρύτητας/Rigidbody).  
**Προαπαιτούμενα:** Βασική C# σύνταξη, έννοια του `GameObject`, κύκλος ζωής MonoBehaviour (`Start`, `Update`).

---

## 1) Τι είναι τα Components;
Κάθε `GameObject` στην Unity αποτελείται από «κομμάτια» (components) που δίνουν συμπεριφορά ή/και εμφάνιση: `Transform`, `Renderer`, `Collider`, `Rigidbody`, `AudioSource`, κ.ά.  
Συνήθως ένα `MonoBehaviour` script αλληλεπιδρά με άλλα components μέσω μεθόδων όπως `GetComponent<T>()`, `TryGetComponent<T>(out T)`, `GetComponentInChildren<T>()`, `GetComponentInParent<T>()`, ή μέσω αναφορών που ορίζουμε στο Inspector.

---

## 2) Πρόσβαση σε components του *ίδιου* GameObject

### 2.1 `GetComponent<T>()` και caching
```csharp
using UnityEngine;

public class SelfComponentsDemo : MonoBehaviour
{
    private Renderer rend;       // MeshRenderer ή SpriteRenderer, ανάλογα με το αντικείμενο
    private Rigidbody rb;        // μπορεί να είναι null, αν δεν υπάρχει

    void Awake()
    {
        // Βρίσκουμε και «κρατάμε» (cache) το component για απόδοση
        rend = GetComponent<Renderer>();
        rb   = GetComponent<Rigidbody>();
    }

    void Start()
    {
        // Ασφαλής χρήση με έλεγχο null
        if (rend != null)
        {
            // Αλλαγή χρώματος (δημιουργεί instance υλικού μόνο για αυτό το αντικείμενο)
            rend.material.color = Color.red;
        }

        if (rb != null)
        {
            // Ελέγχουμε/ορίζουμε βαρύτητα
            rb.useGravity = false;
        }
    }
}
```
**Σημείωση για υλικά:**  
- `renderer.material` ⇒ δημιουργεί instance υλικού μόνο για το συγκεκριμένο αντικείμενο (καλό για runtime αλλαγές χρώματος).  
- `renderer.sharedMaterial` ⇒ μοιρασμένο υλικό (οι αλλαγές επηρεάζουν όλα τα αντικείμενα που το μοιράζονται).

### 2.2 Εμφάνιση/κρύψιμο Renderer
```csharp
public class ToggleRenderDemo : MonoBehaviour
{
    private Renderer rend;

    void Awake()
    {
        rend = GetComponent<Renderer>();
    }

    void Update()
    {
        if (Input.GetKeyDown(KeyCode.H) && rend != null)
        {
            rend.enabled = !rend.enabled; // κρύψε/εμφάνισε
        }
    }
}
```

### 2.3 Προσθήκη component δυναμικά (π.χ. `Rigidbody`)
```csharp
public class AddRigidbodyIfMissing : MonoBehaviour
{
    void Start()
    {
        if (!TryGetComponent<Rigidbody>(out var rb))
        {
            rb = gameObject.AddComponent<Rigidbody>();
            rb.useGravity = true;
            rb.mass = 1f;
        }
    }
}
```

---

## 3) Πρόσβαση σε *άλλα* GameObjects και στα components τους

Υπάρχουν πολλοί τρόποι. Προτιμούμε αναφορές από το Inspector για σαφήνεια/απόδοση. Άλλες επιλογές: `FindWithTag`, `Find`, `FindObjectOfType`, `GetComponentInChildren`, κ.λπ. Προσοχή στο κόστος τους.

### 3.1 Μέσω αναφοράς στο Inspector (προτεινόμενο)
```csharp
public class OtherObjectRef : MonoBehaviour
{
    [SerializeField] private GameObject target; // σύρε εδώ το αντικείμενο
    private Renderer targetRenderer;

    void Awake()
    {
        if (target != null)
            targetRenderer = target.GetComponent<Renderer>();
    }

    public void MakeBlue()
    {
        if (targetRenderer != null)
            targetRenderer.material.color = Color.blue;
    }
}
```

### 3.2 Εύρεση με Tag
1) Ορίζουμε ένα Tag (π.χ. `Target`) στο Inspector του άλλου αντικειμένου.  
2) Κώδικας:
```csharp
public class FindByTagDemo : MonoBehaviour
{
    private Renderer targetRenderer;

    void Start()
    {
        var go = GameObject.FindWithTag("Target");
        if (go != null)
        {
            targetRenderer = go.GetComponent<Renderer>();
            if (targetRenderer != null)
                targetRenderer.material.color = Color.green;
        }
    }
}
```

### 3.3 Εύρεση με όνομα (λιγότερο ασφαλές)
```csharp
void Start()
{
    var go = GameObject.Find("MyCube"); // προσοχή στα ονόματα/ιεραρχία
    if (go != null)
    {
        var rend = go.GetComponent<Renderer>();
        if (rend != null) rend.enabled = false;
    }
}
```

### 3.4 Πρόσβαση σε παιδιά/γονέα
```csharp
void Start()
{
    // Component σε παιδί
    var childRend = GetComponentInChildren<Renderer>();
    // Component σε γονέα
    var parentRb = GetComponentInParent<Rigidbody>();
}
```

---

## 4) Παραδείγματα «όλα μαζί»

### 4.1 Αλλαγή χρώματος με πλήκτρα & toggle ορατότητας άλλου αντικειμένου
```csharp
using UnityEngine;

public class ColorAndVisibilityController : MonoBehaviour
{
    [Header("Target to control")]
    [SerializeField] private GameObject target;

    private Renderer selfRenderer;
    private Renderer targetRenderer;

    void Awake()
    {
        selfRenderer = GetComponent<Renderer>();
        if (target != null) targetRenderer = target.GetComponent<Renderer>();
    }

    void Update()
    {
        // Αλλαγή χρώματος του *δικού μας* αντικειμένου
        if (Input.GetKeyDown(KeyCode.Alpha1) && selfRenderer != null)
            selfRenderer.material.color = Color.red;

        if (Input.GetKeyDown(KeyCode.Alpha2) && selfRenderer != null)
            selfRenderer.material.color = Color.yellow;

        if (Input.GetKeyDown(KeyCode.Alpha3) && selfRenderer != null)
            selfRenderer.material.color = Color.cyan;

        // Toggle ορατότητας του *άλλου* αντικειμένου
        if (Input.GetKeyDown(KeyCode.V) && targetRenderer != null)
            targetRenderer.enabled = !targetRenderer.enabled;
    }
}
```

### 4.2 Προσθήκη/ρύθμιση βαρύτητας σε πολλά αντικείμενα μέσω Tag
```csharp
public class GravityForMany : MonoBehaviour
{
    [SerializeField] private string tagName = "FallingObject";
    [SerializeField] private bool enableGravity = true;
    [SerializeField] private float mass = 1f;

    void Start()
    {
        var all = GameObject.FindGameObjectsWithTag(tagName);
        foreach (var go in all)
        {
            if (!go.TryGetComponent<Rigidbody>(out var rb))
                rb = go.AddComponent<Rigidbody>();

            rb.useGravity = enableGravity;
            rb.mass = mass;
        }
    }
}
```

---

## 5) Καλές πρακτικές & παγίδες
- **Caching**: Κάνε `GetComponent` σε `Awake/Start` και κράτα αναφορά, αντί για συνεχή κλήση στο `Update`.  
- **Έλεγχος null**: Πάντα έλεγξε αν το component υπάρχει πριν το χρησιμοποιήσεις.  
- **`TryGetComponent`**: Αποφεύγει exceptions και είναι λίγο πιο αποδοτικό.  
- **Εύρεση με `Find*`**: Χρήσιμη αλλά πιο βαριά/εύθραυστη. Προτίμησε Inspector αναφορές ή Tags.  
- **Υλικά (Materials)**: Προτίμησε `renderer.material` για runtime αλλαγές χρώματος (δημιουργεί instance).  
- **2D/3D Renderers**: Σε 2D χρησιμοποίησε `SpriteRenderer` (ιδιότητες όπως `color`). Σε 3D συνήθως `MeshRenderer`/`SkinnedMeshRenderer`.  
- **Hierarchy access**: `GetComponentInChildren/Parent` για σύνθετα αντικείμενα.  
- **Ελάχιστες εξαρτήσεις**: Κράτα τον κώδικα χαλαρά δεμένο (π.χ. μέσω Interface ή Events αν μεγαλώσει η πολυπλοκότητα).

---

## 6) Μικρές ασκήσεις

### Άσκηση 1 — Χειρισμός Renderer & Χρώματος
- **Στόχος:** Να αλλάζεις χρώμα και ορατότητα ενός κύβου με πλήκτρα.  
- **Βήματα:**
  1. Δημιούργησε μια Σκηνή με έναν `Cube` (3D Object → Cube).  
  2. Πρόσθεσε ένα νέο script `CubeController.cs` με κώδικα παρόμοιο με το `ColorAndVisibilityController`.  
  3. Με `1/2/3` άλλαζε χρώμα. Με `V` κρύβε/εμφάνιζε.  
  4. **Bonus:** Πρόσθεσε αργή εναλλαγή χρώματος (π.χ. `Color.Lerp` στο `Update`).

### Άσκηση 2 — Προσθήκη/Έλεγχος Βαρύτητας Μαζικά
- **Στόχος:** Να προσθέτεις ή να αφαιρείς βαρύτητα από πολλαπλές σφαίρες ταυτόχρονα.  
- **Βήματα:**
  1. Δημιούργησε 5–10 `Sphere` με Tag `FallingObject`.  
  2. Πρόσθεσε το script `GravityForMany` σε ένα άδειο `GameObject` (`GameObject → Create Empty`).  
  3. Ρύθμισε από το Inspector το `tagName`, το `enableGravity` και τη `mass`.  
  4. **Bonus:** Πρόσθεσε πλήκτρο `G` που εναλλάσσει δυναμικά `useGravity` σε όλα (π.χ. αποθήκευσε τα `Rigidbody` στην αρχή και κάνε toggle).

---

## 7) Checklist κατανόησης
- [ ] Μπορώ να πάρω component του ίδιου GameObject με `GetComponent`/`TryGetComponent`.  
- [ ] Μπορώ να ορίσω αναφορές άλλων GameObjects στο Inspector και να βρω components τους.  
- [ ] Μπορώ να αλλάξω χρώμα/material με ασφάλεια.  
- [ ] Μπορώ να κρύβω/εμφανίζω renderer.  
- [ ] Μπορώ να προσθέτω `Rigidbody` και να ελέγχω `useGravity`.  
- [ ] Γνωρίζω πότε να χρησιμοποιώ Tags, `Find`, Children/Parent lookups και πότε να αποφεύγω κακές πρακτικές.

---

## 8) Συχνά λάθη
- Χρήση `sharedMaterial` αντί για `material` όταν θες τοπική αλλαγή.  
- Συνεχές `GetComponent` στο `Update` χωρίς caching.  
- Εξάρτηση από `GameObject.Find("όνομα")` που αλλάζει εύκολα.  
- Αγνόηση ελέγχων `null` που οδηγεί σε exceptions.

---

Καλή συνέχεια! Στο επόμενο μικρομάθημα θα περάσουμε σε πιο οργανωμένη διαχείριση πολλών αντικειμένων και επικοινωνία μεταξύ scripts.

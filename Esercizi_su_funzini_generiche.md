# Esercizi su Funzioni generiche prove d'esame



# Esercizi — Funzione Generica

---

## Esercizio 1 — `min_element`

**(Funzione generica)** Scrivere l'implementazione dell'algoritmo generico **min_element**, che prende come input una sequenza ed un criterio di ordinamento (un predicato binario) e restituisce l'iteratore all'elemento della sequenza di valore minimo (secondo tale criterio di ordinamento); l'algoritmo si comporta in modo standard nel caso di sequenza vuota. Elencare in modo esaustivo i requisiti imposti dall'implementazione sui parametri di tipo e sugli argomenti della funzione. In particolare, individuare le categorie di iteratori che non possono essere utilizzate per istanziare il template, motivando la risposta.

```cpp
template <typename ForwardIt, typename Compare>
ForwardIt min_element(ForwardIt first, ForwardIt last, Compare comp)
{
    if (first == last) // sequenza vuota
        return last;

    ForwardIt min = first; // iteratore al minimo corrente
    ++first;

    for (; first != last; ++first) {
        if (comp(*first, *min)) // se *first è "più piccolo" di *min
            min = first;
    }

    return min;
}
```

Dall'implementazione si vede che `ForwardIt` deve supportare:

- costruzione e copia (passato e restituito per valore)
- confronto: `first == last`, `first != last`
- avanzamento: `++first`
- dereferenziazione in lettura: `*first`

Quindi `ForwardIt` deve essere **almeno un Forward Iterator**.

In particolare:

- **NON basta un OutputIterator** (non supporta lettura né confronto).
- **Un InputIterator non è sufficiente**, perché l'algoritmo memorizza un iteratore (`min`) e continua a scorrere la sequenza: questo richiede iteratori **multipass** (garanzia fornita solo dai forward iterator e superiori).

Sono invece ammessi:

- `ForwardIterator`
- `BidirectionalIterator`
- `RandomAccessIterator`

---

## Esercizio 2 — `find_if`

**(Funzione generica)** Definire la funzione generica **find_if** che prende in input una sequenza ed un predicato unario e restituisce in output un iteratore che individua il primo elemento della sequenza che soddisfa il predicato.

```cpp
template <typename Iter, typename Pred>
Iter find_if(Iter first, Iter last, Pred pred)
{
    for (; first != last; ++first) {
        if (pred(*first)) // se *first rispetta il predicato
            return first;
    }

    return last;
}
```

---

## Esercizio 3 — `any_of`

**(Funzione generica)** Definire la funzione generica **any_of** che prende in input una sequenza ed un predicato unario e restituisce in output un booleano: la funzione restituisce il valore `true` se e solo se almeno uno degli elementi della sequenza soddisfa il predicato. Scrivere inoltre una funzione che prende in input un vettore di numeri interi e, usando la funzione **any_of** appena definita, controlla se almeno uno dei numeri contenuti è negativo.

```cpp
template <typename Iter, typename Pred>
bool any_of(Iter first, Iter last, Pred pred) {
    for (; first != last; ++first) {
        if (pred(*first)) // se *first rispetta il predicato
            return true;
    }

    return false;
}

#include <vector>

bool has_negative(const std::vector<int>& v)
{
    return any_of(
        v.begin(), v.end(),      // definiscono l'intervallo [begin, end) su cui lavorerà any_of
        [](int x) { return x < 0; }  // prendo un intero x e restituisco true se x è minore di zero
    );
}
```

---

## Esercizio 4 — `replace`

**(Funzione generica)** Definire la funzione generica **replace** che, presi in input una sequenza e due valori `old_value` e `new_value` di tipo generico `T`, rimpiazza nella sequenza ogni elemento equivalente a `old_value` con una copia di `new_value` (nota bene: l'algoritmo non produce una nuova sequenza, ma modifica direttamente la sequenza fornita in input). Elencare i requisiti imposti dall'implementazione sui parametri della funzione.

```cpp
template <typename Iter, typename T>
void replace(Iter first, Iter last, const T& old_value, const T& new_value) {
    for (; first != last; ++first) {
        if (*first == old_value)
            *first = new_value;
    }
}
```

Categorie di iteratori:

- **Input Iterator** ❌ → non basta, perché non puoi scrivere (`*it = ...`)
- **Output Iterator** ❌ → non basta, perché bisogna leggere e confrontare
- **Forward Iterator** ✅ → necessario, permette leggere, scrivere e scorrere
- **Bidirectional / Random Access Iterator** ✅ → vanno bene, sono più forti

---

## Esercizio 5 — `transform`

**(Funzione generica)** Si forniscano il prototipo e l'implementazione della funzione generica **transform** i cui parametri specificano: (a) una sequenza di ingresso; (b) una sequenza di uscita; (c) una funzione unaria applicabile agli elementi della sequenza in ingresso e che restituisce un elemento assegnabile alla sequenza di uscita. La funzione **transform** scrive, nella sequenza di uscita, i risultati dell'applicazione della funzione unaria ad ogni elemento della sequenza in ingresso.

```cpp
template <typename InputIter, typename OutputIter, typename UnaryFunc>
OutputIter transform(InputIter first, InputIter last, OutputIter d_first, UnaryFunc unary)
{
    for (; first != last; ++first, ++d_first) {
        *d_first = unary(*first);
    }

    return d_first; // restituisce l'iteratore alla fine della sequenza di output
}
```

---

## Esercizio 6 — `contains` (implementazione diretta)

Definire la funzione generica **contains** che prende in input due sequenze (non necessariamente dello stesso tipo) e restituisce in output un booleano. La funzione restituisce il valore `true` se e solo se ogni elemento della seconda sequenza compare, in una posizione qualunque, nella prima sequenza. Nota: si richiede di implementare la funzione senza fare ricorso ad altri algoritmi generici della libreria standard.

```cpp
template <typename InputIter1, typename InputIter2>
bool contains(InputIter1 first, InputIter1 last, InputIter2 first2, InputIter2 last2)
{
    for (; first2 != last2; ++first2) {
        bool contiene = false;
        for (InputIter1 it = first; it != last; ++it) {
            if (*it == *first2) {
                contiene = true;
                break;
            }
        }

        if (!contiene)
            return false;
    }

    return true;
}
```

---

## Esercizio 7 — `contains` (con `std::find`)

**(Funzione generica)** Definire la funzione generica **contains** che prende in input due sequenze, non necessariamente dello stesso tipo, e restituisce in output un booleano: la funzione restituisce il valore `true` se e solo se ogni elemento della seconda sequenza compare, in una posizione qualunque, nella prima sequenza. Implementare la funzione usando, in maniera appropriata, l'algoritmo generico **std::find**; elencare quindi i requisiti imposti dall'implementazione sui parametri della funzione.

```cpp
#include <algorithm> // std::find

template <typename InputIter1, typename InputIter2>
bool contains(InputIter1 first1, InputIter1 last1,
              InputIter2 first2, InputIter2 last2)
{
    for (; first2 != last2; ++first2) {
        if (std::find(first1, last1, *first2) == last1) {
            return false; // elemento non trovato nella prima sequenza
        }
    }

    return true; // tutti gli elementi trovati
}
```

Iteratori minimi richiesti: **InputIterator per entrambe le sequenze**.
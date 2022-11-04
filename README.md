# LP25 - TP C n°2

Ce TP est une introduction à l'usage des fichiers en C. Vous y verrez comment lire et écrire séquentiellement des fichiers textes et binaires. Dans la session suivante, nous aborderons la lecture et l'écriture non séquentielles (i.e. sans commencer du début en progressant vers la fin), ainsi que les verrous sur fichier.

## Exercice 1 - Lecture de fichiers texte

Cet exercice consiste à lire dans un fichier texte et à afficher ce qui est lu. Le programme pour réaliser l'exercice s'appellera `my-cat.c`. Vous pouvez lui joindre des fichiers entête et code additionnels à la condition de fournir également un Makefile pour pouvoir compiler toute votre production de ce TP.

Le nom du fichier à lire est passé en argument du programme.

Pour rappel, utiliser un fichier se fait de la manière suivante :

```c
#include <stdio.h>

#define BUFFER_SIZE 150

int main() {
	FILE *f = fopen("path_to_file", "r"); // Open in text mode, for reading
	if (!f) return 1;
	char buffer[BUFFER_SIZE];
	while (fgets(buffer, BUFFER_SIZE, f)) {
		// Do something with buffer content
	}
	fclose(f); // Do not forget that to free file handle resources
	return 0;
}
```

### Fonctions utiles

Vous utiliserez pour cet exercice :

- `fopen` pour l'ouverture du fichier
- `fclose` pour sa fermeture
- `fgets` ou `fscanf` pour la lecture

## Exercice 2 - Écriture de fichiers texte

Cet exercice sera nommé `write-text.c` et consistera à écrire dans un fichier texte. Le programme prend en argument un entier `s` au moins égal à 4 (si le paramètre manque, ou est invalide, la valeur 4 sera utilisée) pour écrire dans le fichier de sortie un losange de `1 + 2 * (s - 1)` de haut et de large, de la manière suivante (exemple pour la valeur 4) :

```text
   *
  ***
 *****
*******
 *****
  ***
   *
```

### Fonctions utiles

Vous utiliserez pour cet exercice (en plus des fonctions de l'exercice 1) :

- `fputs` ou `fprintf` pour l'écriture

## Exercice 3 - Lecture de fichiers binaires

### Ouverture avec FILE

L'ouverture d'un fichier en lecture binaire se fait avec le mode `"rb"`, et la lecture avec la fonction `fread`.

Dans cet exercice, dont le fichier source sera nommé `binary-read.c`, un fichier contenant les représentations binaires d'entiers (`int`) est fourni. Vous devrez le lire et stocker ses données dans une liste doublement chaînée dans l'ordre croissant.

La liste chaînée est définie comme suit :
```c
typedef struct _list_element {
	int value;
	struct _list_element *next; // Points to next element, NULL if this is the last one
	struct _list_element *prev; // Points to previous element, NULL if this is the first one
} list_element_t;

typedef struct {
	list_element_t *head; // Points to first element, NULL if list is empty
	list_element_t *tail; // Points to last element, NULL if list is empty
} list_t;

/* All functions below: you must implement them */
void add_int_to_list(int value, list_t *list); // Add in order
void clear_list(list_t *list); // Free all elements
void display_list(list_t *list); // Display the list

/* List usage will be something like: */
int main(int argc, char *argv[]) {
	// Open file
	list_t numbers = {.head=NULL, .tail=NULL,}; // Init empty list
	while (/* Iterate over file content to get int values */) {
		add_int_to_list(a_value, &numbers);
	}
	display_list(&numbers); // Display our numbers in order
	clear_list(&numbers); // Free memory allocated
	return 0;
}
```

La fonction `fread(void *restrict ptr, size_t taille, size_t nbelem, FILE *restrict flux)` prend plusieurs paramètres pour son fonctionnement :

- `ptr` est un pointeur vers un buffer qui contiendra le résultat de la lecture effectuée par l'appel à `fread`,
- `taille` est un entier qui détermine la taille d'un élément lu dans le fichier. On l'obtient souvent avec l'instruction `sizeof`,
- `nbelem` est le nombre d'éléments que l'on demande de lire. `ptr` doit donc faire une taille d'au moins `taille * nbelem` pour permettre de stocker le résultat de la lecture,
- `flux` est le pointeur de fichier dans lequel les données vont être lues.

`fread` retourne le nombre d'éléments lus. Plus d'information sur le fonctionnement de cette fonction sont disponibles avec la commande `man fread`.

### Alternative : open/read

Il existe une possibilité offerte par Linux pour manipuler des fichiers avec des descripteurs (de type `int`) similaires à tous les descripteurs utilisés par Linux (par exemple pour les communications réseau avec `socket`, vus dans l'UV [RI52](https://guideuv.utbm.fr/#!/Fr/2022/GI/RI52) de la filière réseaux et infrastructure de la FISE informatique).

Deux façons de récupérer ce descripteur existent :

- soit ouvrir un fichier avec la fonction `open` qui renvoie un descripteur de fichier (voir `man open`)
- soit en obtenant le descripteur associé à un `FILE*` grâce à la fonction `fileno` (voir `man fileno`)

Une fois le descripteur obtenu, on utilise la fonction `read(int fd, void *buf, size_t count)` (cf. `man 2 read` pour le manuel détaillé), qui prend 3 paramètres :

- `fd` le descripteur de fichier du fichier dans lequel on va lire,
- `buf` le buffer dans lequel sera stocké le résultat de la lecture,
- `count` le nombre maximum d'octets lus dans le fichier. `buf` doit être d'une longueur au moins égale à `count`.

`read` renvoie le nombre effectif d'octets lus. Un appel comme :

```c
int value;
ssize_t bytes_read = read(my_fd, &value, sizeof(int));
```
permet de lire un nombre d'octets égal à la taille d'un entier, et de stocker le résultat dans l'entier `value`. C'est typiquement une instruction qui permettrait de lire un entier dans un fichier binaire.

### Fonctions utiles

Vous utiliserez pour cet exercice (en plus des fonctions des exercices 1 et 2) :

- `open` pour ouvrir un fichier en tant que descripteur de fichier de type `int`
- `fileno` pour obtenir le descripteur de fichier d'un `FILE*`
- `read` ou `fread` pour lire dans le fichier

## Exercice 4 - Écriture de fichiers binaires

L'ouverture d'un fichier binaire en écriture se fait avec le mode `"wb"` et l'écriture se fait avec la fonction `fwrite`.

Dans cet exercice, dont le code est nommé `binary-write.c`, vous reprendrez le code de l'exercice 3 pour écrire dans un fichier (différent de la source des données) le résultat du tri des données. Vous utiliserez pour cela soit un `FILE` et `fwrite`, soit un descripteur de fichier et les fonctions `open` et `write`.

Le prototype de la fonction à ajouter aux listes chaînées est le suivant :
```c
void save_to_file(char *filename, list_t *list);
```

### Fonctions utiles

Vous utiliserez pour cet exercice (en plus des fonctions des exercices 1 et 2) :

- `open` pour ouvrir un fichier en tant que descripteur de fichier de type `int`
- `fileno` pour obtenir le descripteur de fichier d'un `FILE*`
- `write` ou `fwrite` pour lire dans le fichier

## Bilan

Dans ce TP, vous avez découvert l'utilisation séquentielle des fichiers, que ce soit en lecture ou en écriture. Cette utilisation vous a permis de comprendre le flux de manipulation du fichier suivant les 3 étapes chronologiques suivantes :

- Ouverture du fichier : `fopen` ou `open` (ou `fopen` puis `fileno`)
- Utilisation
	- En lecture : `fgets`, `fscanf`, `fread` ou `read`
	- En écriture : `fputs`, `fprintf`, `fwrite` ou `write`
- Fermeture : `fclose` ou `close`

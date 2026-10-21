# PA 8: The VTable Dispatcher

## 1. Building the table

Use slot 0 for `speak` and slot 1 for `name` in all three classes. Each entry points to the method implementation shown below.

| Class | Slot 0: speak | Slot 1: name |
|---|---|---|
| Animal | Animal.speak | Animal.name |
| Dog | Dog.speak (overrides Animal.speak) | Animal.name (inherited unchanged from Animal) |
| Puppy | Dog.speak (inherited unchanged from Dog) | Puppy.name (overrides inherited Animal.name) |

Conceptual VTable arrays:

```text
Animal: [Animal.speak, Animal.name]
Dog:    [Dog.speak,    Animal.name]
Puppy:  [Dog.speak,    Puppy.name]
```

Dog's name entry is the exact same Closure object as Animal's name entry. Puppy's speak entry is the exact same Closure object as Dog's speak entry. The builder copies the entry dictionary, not the inherited closures. Updating a subclass table therefore does not change its parent's table. In dispatcher.py, the slot names are dictionary keys; the numeric slots above describe the conceptual array layout.

## 2. Static vs. dynamic dispatch

Conceptual USILang snippet (the type annotation expresses the declared base type):

```text
var pet: Animal = Animal();
pet = Puppy();
pet.speak();
```

A statically dispatched call based on the declared Animal type resolves to Animal.speak and returns "...". A dynamically dispatched call uses the actual Puppy object and returns "Woof" from Dog.speak.

The dynamic call consults Puppy's VTable, reads slot 0 (speak), and finds the pointer to Dog.speak. Puppy inherits that slot unchanged from Dog. Using Animal's table would select Animal.speak instead, ignoring the override inherited by the runtime Puppy instance. The snippet describes dispatch semantics; the supplied Python harness does not implement a static type system or parse class syntax.

## 3. Polymorphism trace

For the list [Animal(), Dog(), Puppy()], a loop calling name() produces:

| Instance | Runtime VTable consulted | Slot | Resolved implementation | Result |
|---|---|---|---|---|
| Animal() | Animal | 1: name | Animal.name | Animal |
| Dog() | Dog | 1: name | Animal.name | Animal |
| Puppy() | Puppy | 1: name | Puppy.name | Puppy |

```text
Animal
Animal
Puppy
```

Dog does not override name(), so its table already contains Animal.name copied by reference during table construction. The inheritance fall-through is handled when building the table; dispatch does not need to search the parent again. Puppy replaces this inherited name entry with Puppy.name.

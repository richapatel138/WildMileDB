# WildMileDB - Vertebrate
This repository serves to explain how to build refrence databases using CRABS for Illinois vertebrate and mussel species, with the generated databases attached along with other helpful files. This specific README files focuses on how to curate the vertebrate database.

### Making a List of Species of Interest

The first (and most tedious) step of creating the refrence database if creating a list of species of intrest. This is what we will use to subset the sequences we get from public repositories, and it allows for a more focused scope. There are many different ways that you can choose to go about this process. For my project, I wanted to focus on vertebrate species found in Illinois, encompasing fish, mammals, bird, amphibians, and reptiles.

The most comprehensive approach I found was utilizing the Illinois Natural History Survey (INHS), and going to the respective website to collect species found in Illinois. I used all the species found on each website to create the refrence subset list. 

* [Fish](https://fish.inhs.illinois.edu/illinois-species-list/)
* [Mammals](https://mammals.inhs.illinois.edu/mammals-of-illinois/)
* [Birds](https://bird.inhs.illinois.edu/birds-of-illinois-checklist/?_gl=1*1vx9wqg*_ga*MjU0MzE3MTI4LjE3NTc0Mzc4MTM.*_ga_8XRWZCXCM7*czE3NTc0Mzc4MTMkbzEkZzAkdDE3NTc0Mzc4MjEkajUyJGwwJGgw)
* [Amphibians & Reptiles](https://herpetology.inhs.illinois.edu/species-lists/ilspecies/)

CRABS requires this list of species to be in a line seperated `.txt` file, with NO header.

After you have a list of species of interest, you will also want to add common contaminants to the subset list, these include:
* Homo sapiens (Human)
* Gallus gallus (Chicken)
* Bos taurus (Cow)
* Canis lupus familiaris (Dog)
* Equus caballus (Horse)
* Thunnini (Tuna)
* Felis catus (Cat)
* Capra hircus (Goat)

Additionally, depending on what types of positive controls are used in your project design, you will also want to add those to the list as well. For us, they were:
* Negaprion (Lemon sharks)
* Squatina (Angel sharks)

The complete list of vertebrates for Illinois using the above info is included in this repository, here. 







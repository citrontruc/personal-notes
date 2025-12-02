# Faire en sorte que le github suive le projet Unity

1) Créez un projet github et demander à inclure comme template de .gitignore le template Unity.
1) Créez un projet dans unity. Sauvegardez le là où vous le voulez.
1) Avec l'interface de commande, se rendez-vous dans la directory de votre projet Unity et tapez les commandes suivantes :
    ```bash
    git init
    git remote add origin <repo-url>
    git fetch
    git checkout main
    ```
    Cela va copier le github dans la direction de votre projet Unity. Normalement, si vous avez le bon gitignore, la majorité des dossiers de Unity seront ignorés.
1) Si vous travaillez avec VSCode, rajouter au fichier gitignore la ligne .vscode/


# Rajouter des pre-commit hook

Pour aller plus loin, on souhaite faire notre setup de tel sorte à ce qu'un linter tourne automatiquement pour mettre en forme notre code. La partie suivante nécessite l'installation de dotnet (testé avec dotnet 8 et 9).
1) Afin d'installer des outils, nous avons besoin d'ajouter un manifeste à notre projet. Cela est possible avec la commande suivante :
    ```bash
    dotnet new tool-manifest
    ```
1) Rajoutez C-sharpier avec la commande suivante :
    ```bash
    dotnet tool install csharpier
    ```
    Il est maintenant possible de lancer C-sharpier avec la commande suivante :
    ```bash
    dotnet csharpier format .
    ```
1) Pour que le linter se lance automatiquement, créez un fichier bash (.sh) avec le contenu suivant :
    ```bash
    #!/bin/sh
    echo "Pre-commit hook is running"
    dotnet tool run csharpier format .
    if [ $? -ne 0 ]; then
    echo "CSharpier formatting failed. Please fix formatting before committing."
    exit 1
    fi
    ```
    Nous allons l'appeler pre-commit.sh. Créez un fichier exécutable à partir de ce fichier avec la commande suivante :
    ```bash
    chmod +x pre-commit.sh
    ```
1) Déplacez le fichier exécutable dans le dossier .git/hooks/ (il s'agit d'un dossier invisible, vous aurez peut-être besoin de modifier les paramètres de vos dossiers pour le rendre visible).

# Ajouter des tests

[Lien du tutoriel microsoft](https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-csharp-with-mstest) 

Notre code est enregistré sur git et nous lançons automatiquement notre linter. Notre prochaine étape est de rajouter des tests à notre code.
1) Créez un dossier Game.Test
1) Rajoutez un projet de tests en lançant la commande suivante :
    ```bash
    dotnet new MSTest
    ```
1) Ajoutez une référence au projet initial avec la commande suivante :
    ```bash
    dotnet reference add ../PrimeService/PrimeService.csproj
    ```
1) Rajoutez une solution à votre projet de test. Pour cela, lancer la commande suivante :
    ```bash
    dotnet sln add ./PrimeService.Tests/PrimeService.Tests.csproj
    ```
    Vous pouvez alors lancer les fichiers de tests en vous trouvant dans la direction principale et en lançant la commande suivante :
    ```bash
    dotnet test
    ```

# Ajouter des github actions

Dans cette partie-là, nous allons mettre en place une github action afin de lancer automatiquement nos tests.
1) Créez un dossier .github et ajoutez dedans les deux sous-directions suivantes :
    - actions (dossier qui va contenir des actions réutilisables)
    - workflows (dossier qui va contenir les suites d'action à effectuer)
1) Nous allons créer une action qui lance les tests. Dans la direction .github/actions/run-test, créez un fichier action.yml avec le contenu suivant :
    ```yml
    name: "Run tests"
    description: "A composite action to run all our tests and check that we don't have problems in our git."

    inputs:
    dotnet-version:
        description: "The dotnet version to use to run our tests"
        required: true

    runs:
    using: "composite"

    steps:
        - name: Checkout repository
        uses: actions/checkout@v4
        
        - name: Install .NET Core
        uses: actions/setup-dotnet@v4
        with:
            dotnet-version: ${{ inputs.dotnet-version }}

        - name: Restore dependencies
        run: dotnet restore
        shell: bash

        - name: Build
        run: dotnet build --no-restore
        shell: bash

        - name: Run tests
        run: dotnet test --no-build --verbosity normal
        shell: bash

        - id: run-script
        run: echo "result=Success" >> $GITHUB_OUTPUT
        shell: bash
    ```
1) Dans le dossier workflows, créer un fichier test.workflow.yml avec le contenu suivant :
    ```yml
    name: Run tests

    on:
    push:

    jobs:
    test:
        runs-on: ubuntu-latest

        steps:
        - name: Checkout repository
        uses: actions/checkout@v4

        - name: Run tests action
        uses: ./.github/actions/run-tests
        with:
            dotnet-version: '8.0.x'
    ```
    Changez les paramètres pour correspondre à la version de dotnet que vous souhaitez utiliser.

Normalement, le workflow devrait se lancer lors de chaque push de code sur n'importe quelle branche.
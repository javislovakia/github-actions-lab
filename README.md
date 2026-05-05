# 1. Workflow CI para el proyecto de frontend
Vamos a crear un workflow en Github para el proyecto hangman-front donde usamos la integración continua para automatizar el *build* y los test unitarios del proyecto cuando hay cambios (push o pull requests) en la rama *main*, y en concreto en la carpeta *hangman-front*.

Para ello, creo el siguiente archivo en la carpeta raiz del proyecto: **.github/workflows/ci-frontend.yaml**

Y añado el siguiente contenido:

```yaml
name: Integración Continua

on:
  push:
    branch: [ main ]
    paths: [ 'hangman-front/**']
  pull_request:
    branches: [ main ]
    paths: [ 'hangman-front/**']

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v6
      - name: Set up Node.js version
        uses: actions/setup-node@v6
        with:
          node-version: 18
      - name: Build
        working-directory: ./hangman-front
        run: |
          npm ci
          npm run build
  
  test:
    runs-on: ubuntu-latest
    needs: build

    steps:
      - name: Checkout
        uses: actions/checkout@v6
      - name: Set up Node.js version
        uses: actions/setup-node@v6
        with:
          node-version: 18
      - name: Unit tests
        working-directory: ./hangman-front
        run: |
          npm ci
          npm run test
```

El fichero yaml especifica cuándo debe ejecutarse el workflow en la sección *on:*, y que acciones va a hacer en la sección *jobs:*

Usamos *actions* de Github para llevar el código del proyecto a la máquina virtual donde se ejecuten el build y los tests. De la forma en la que están configurados, se ejecutará primero el build, y si tiene éxito, se lanzará el test.

Ahora, para probarlo, simplemente hay que hacer un commit y un push a github. Para que funcione hay que hacer algún cambio en alguno de los ficheros que están dentro de la carpeta *hangman-front*, si no hacemos esto no se cumplirán las condiciones para que se lance el workflow. Esto lo hemos especificado con *path:* dentro del fichero yaml.

En la primera ejecución del workflow me ha fallado el job test. Mirando los logs dentro del workflow veo el siguiente error:

```
FAIL src/components/start-game.spec.tsx
  StartGame component specs
    ✕ should display a list of topics (69 ms)

  ● StartGame component specs › should display a list of topics

    expect(received).toHaveLength(expected)

    Expected length: 1
    Received length: 2
    Received array:  [<li>topic A</li>, <li>topic B</li>]
```

Para intentar solucionarlo cambio la línea 16 del archivo */hangman.front/src/components/start-game.spec-tsx* por la siguiente:

```typescript
expect(items).toHaveLength(2);
```

Simplemente aumento la longitud experada de 1 a 2.

Después de este cambio hago el push otra vez y esta vez el workflow se ejecuta correctamente.

![CI workflows](./capturas/c1-ciworkflows.png)


# 2. Workflow CD para el proyecto de frontend


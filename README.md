# Dog Shelter Application
<a href="https://www.hyperledger.org"><img src="https://img.shields.io/badge/blockchain-Hyperledger%20Fabric-blue.svg" alt="current version" /></a>
<a href="https://www.hyperledger.org/projects/composer"><img src="https://img.shields.io/badge/tool-Hyperledger%20Composer-blueviolet.svg" alt="current version" /></a>
<a href="https://www.github.com/a01334390/dogShelter"><img src="https://img.shields.io/badge/build-passing-success.svg" alt="current version" /></a>
<a href="https://www.github.com/a01334390/dogShelter"><img src="https://img.shields.io/github/repo-size/a01334390/dogShelter.svg" alt="size"/> </a>

Aplicacion para el control y monitoreo de perros en albergues antes, durante y después de ser adoptados. Trabajo para la clase de Aplicaciones en la Nube, semana 8

## Pasos para la práctica
### Paso 1: Entrar a la página de Composer Playground
Entrar a [esta](https://composer-playground.mybluemix.net/) página para empezar la práctica

![alt Pagina de Hyperledger Playground](https://raw.githubusercontent.com/a01334390/dogShelter/master/pictures/playground-step1.png)
### Paso 2: Crear una nueva red de negocio
Para empezar a escribir el modelo de la red de mascotas, tendremos que seleccionar crear una nueva red vacía de negocios (empty-business-network). Escribiremos lo siguiente en los siguientes parámetros:

```
Give your new business network a name: dog-shelter
```

```
Describe what your Business Network will be used for: Tracking dogs through the process of being accepted by a shelter and being adopted by someone
```

```
Give the network admin card that will be created a name: 
```

![alt Pagina de Hyperledger Playground - Paso 1](https://raw.githubusercontent.com/a01334390/dogShelter/master/pictures/playground-step2.png)

### Paso 3: 

Crea un nuevo archivo .cto para crear modelos. En este archivo, crearemos:
#### Participantes

| Nombre        | Tipo                             | Uso                                        |
| ------------- |:--------------------------------:| ------------------------------------------:|
| Person        | Abstract Participant             | Informacion general para los participantes |
| Shelter       | Participant extending Person     |   Participante encargado del albergue      |
| Adopter       | Participant extending Person     |    Participant encargado de la adopcion    |

#### Modelos

| Nombre        | Tipo                             | Uso                                        |
| ------------- |:--------------------------------:| ------------------------------------------:|
| Dog           | Asset                            | Describe al Asset Perro                    |

#### Transacciones

| Nombre         | Tipo                             | Uso                                                   |
| --------------:|:--------------------------------:| ----------------------------------------------------: |
| shelterProcess | Transaction(Dog)                 | Permite pasar al perro por el proceso del albergue    |
| adoptDog       | Transaction(Dog, Adopter)        | Permite que el perro sea adoptado por un participante |

#### Eventos

_Coming Soon_

### Otros

| Nombre        | Tipo                             | Uso                                         |
| ------------- |:--------------------------------:| -------------------------------------------:|
| STATUS        | ENUM                             | Indicador del status del perro en el proceso|

Al final, el código debe verse de la siguiente manera:

``` gherkin
namespace org.example.basic

enum Status {
  o FOUND
  o DECONTAMINATION
  o SHELTERED
  o ADOPTED
}

asset Dog identified by dogId {
  o String dogId
  o String name 
  o Integer age default=3
  o DateTime foundOn optional
  o String breed
  o Status status
  --> Person owner
}

abstract participant Person identified by personId {
	o String personId
    o String name
    o String street1
    o String street2
    o String country
}

participant Shelter extends Person { }
participant Adopter extends Person { }

transaction shelterProcess {
	--> Dog dog
}

transaction adoptDog {
	--> Dog dog
  --> Adopter adopter
}
```

### Paso 4: Escribir la lógica de negocio

Crear un nuevo archivo .js de Script. En este, tendremos que escribir el siguiente código:
**NOTA** En Hyperledger Composer, los comentarios **SI** se usan durante el proceso de compilación.

```javascript
/* global getAssetRegistry getFactory emit */

/**
 * Sample transaction processor function.
 * @param {org.example.basic.shelterProcess} tx The sample transaction instance.
 * @transaction
 */
async function shelterProcess(tx) {  // eslint-disable-line no-unused-vars
  	const currentStatus = tx.dog.status
    switch(currentStatus){
      case "FOUND":
        tx.dog.status = "DECONTAMINATION"
        break;
      case "DECONTAMINATION":
        tx.dog.status = "SHELTERED"
        break;
      default:
        console.log("Either not defined by this chaincode version or no longer possible to scalate")
        break;
    }
                    
    // Get the asset registry for the asset.
    const assetRegistry = await getAssetRegistry('org.example.basic.Dog');
    // Update the asset in the asset registry.
    await assetRegistry.update(tx.dog);
}

/**
 * Sample transaction processor function.
 * @param {org.example.basic.adoptDog} tx The sample transaction instance.
 * @transaction
 */
async function adoptDog(tx) {  // eslint-disable-line no-unused-vars
  
  // Change a dog's owner
  tx.dog.owner = tx.adopter
  tx.dog.status = "ADOPTED"
  // Get the asset registry
  const assetRegistry = await getAssetRegistry('org.example.basic.Dog')
  //Update the asset
  await assetRegistry.update(tx.dog)

}

```

### Paso 5: Escribir la lista de Acceso
En este paso, crearemos un nuevo archivo .acl en donde escribiremos lo siguiente:

```gherkin
rule Default {
    description: "Allow all participants access to all resources"
    participant: "ANY"
    operation: ALL
    resource: "org.example.basic.**"
    action: ALLOW
}

rule SystemACL {
    description:  "System ACL to permit all access"
    participant: "org.hyperledger.composer.system.Participant"
    operation: ALL
    resource: "org.hyperledger.composer.system.**"
    action: ALLOW
}

rule NetworkAdminUser {
    description: "Grant business network administrators full access to user resources"
    participant: "org.hyperledger.composer.system.NetworkAdmin"
    operation: ALL
    resource: "**"
    action: ALLOW
}

rule NetworkAdminSystem {
    description: "Grant business network administrators full access to system resources"
    participant: "org.hyperledger.composer.system.NetworkAdmin"
    operation: ALL
    resource: "org.hyperledger.composer.system.**"
    action: ALLOW
}
```

### Paso 5: Desplegar y disfutar!

# Diagrames de casos d'us del protocol


#### Registre de client al servidor (èxit)

El següent flux mostra el funcionament correcte del registre d'un client al servidor.

```mermaid
sequenceDiagram
    participant C as Client
    participant S as Servidor

    C->>S: CLIENT_REGISTER (nickname)
    S->>S: Generar Client ID
    S->>C: CLIENT_REGISTER_RESPONSE (Status: 0x00, Client ID, Chunk Size)

```


#### Registre de client al servidor (error)

El següent flux mostra el funcionament quan el registre del client falla.

```mermaid
sequenceDiagram
    participant C as Client
    participant S as Servidor

    C->>S: CLIENT_REGISTER (nickname)
    S->>S: Error al registrar (ex: nickname duplicat)
    S->>C: CLIENT_REGISTER_RESPONSE (Status: 0x01, Error)

```


#### Anunci de fitxers

El següent flux mostra el funcionament de l'anunci de fitxers disponibles al servidor.

```mermaid
sequenceDiagram
    participant C as Client
    participant S as Servidor

    C->>S: FILE_ANNOUNCE (File Count, File 1, File 2, ...)
    S->>S: Actualitzar registre de fitxers del client
    S->>C: ACK

```

#### Cerca de fitxers

El següent flux mostra el funcionament de la cerca de fitxers al servidor.

```mermaid
sequenceDiagram
    participant C as Client
    participant S as Servidor

    C->>S: SEARCH_REQUEST (query)
    S->>S: Cercar fitxers que coincideixin amb la query
    S->>C: SEARCH_RESPONSE (Result Count, File 1 + Client IDs, File 2 + Client IDs, ...)

```

#### Flux de descàrrega de fitxer (pel Client 1 del Client 2) 

El següent flux mostra el funcinament correcte de descarrega d'un fitxer entre un client i un altre.

```mermaid

sequenceDiagram
    participant C1 as Client 1 (Sol·licitant)
    participant S as Servidor
    participant C2 as Client 2 (Origen)

    C1->>S: DOWNLOAD_REQUEST (fitxer)
    
    S->>S: Generar Transfer ID
    S->>C2: SERVER_FILE_REQUEST (fitxer, Transfer ID, Client ID C1)
    C2->>S: SERVER_FILE_RESPONSE (Status: 0x00, Transfer ID, File Info)

    S->>C1: DOWNLOAD_RESPONSE (Status: 0x00, disponible + Transfer ID + File Info)

    C1->>S: CHUNK_REQUEST (Transfer ID, chunk 0)
    S->>C2: CHUNK_REQUEST (Transfer ID, chunk 0)
    C2->>S: CHUNK_RESPONSE (Transfer ID, chunk 0, dades 0)
    S->>C1: CHUNK_RESPONSE (Transfer ID, chunk 0, dades 0)

    C1->>S: CHUNK_REQUEST (Transfer ID, chunk 1)
    S->>C2: CHUNK_REQUEST (Transfer ID, chunk 1)
    C2->>S: CHUNK_RESPONSE (Transfer ID, chunk 1, dades 1)
    S->>C1: CHUNK_RESPONSE (Transfer ID, chunk 1, dades 1)

    C1->>S: HASH_VERIFY (hash correcte)
    S->>C1: ACK
    S->>C2: ACK (transferència completada)

```
#### Flux de fitxer no trobat (Client 1 del Client 2) 

El següent flux mostra el funcionament incorrecte degut a que el fitxer no es troba a l'origen, degut a diversos motius: el fitxer no es troba o el client s'ha desconectat.

```mermaid
sequenceDiagram
    participant C1 as Client 1 (Sol·licitant)
    participant S as Servidor
    participant C2 as Client 2 (Origen)

    C1->>S: DOWNLOAD_REQUEST (fitxer, Source Client ID)
    
    S->>S: Cercar fitxer al registre
    S->>S: Trobar client origen (C2)
    S->>S: Generar Transfer ID
    
    S->>C2: SERVER_FILE_REQUEST (fitxer, Transfer ID, Client ID C1)
    
    alt Fitxer no trobat al client origen
        C2->>S: SERVER_FILE_RESPONSE (Status: 0x01, Transfer ID, Fitxer no trobat)
        S->>C1: DOWNLOAD_RESPONSE (Status: 0x01, Fitxer no trobat)
    else Client origen desconectat o no disponible
        Note over S: Timeout o error de connexió
        S->>C1: DOWNLOAD_RESPONSE (Status: 0x02, Client origen no disponible)
    else Fitxer no trobat al registre del servidor
        Note over S: El fitxer no existeix al registre
        S->>C1: DOWNLOAD_RESPONSE (Status: 0x01, Fitxer no trobat)
    end

```

#### Flux de fitxer no verificat correctament
El següent flux mostra el funcionament incorrecte degut a que no es pot fer la verificació del hash del fitxer

```mermaid
sequenceDiagram
    participant C1 as Client 1
    participant S as Servidor
    participant C2 as Client 2

    C1->>S: DOWNLOAD_REQUEST (fitxer)
    
    S->>S: Generar Transfer ID
    S->>C2: SERVER_FILE_REQUEST (fitxer, Transfer ID, Client ID C1)
    C2->>S: SERVER_FILE_RESPONSE (Status: 0x00, Transfer ID, File Info)

    S->>C1: DOWNLOAD_RESPONSE (Status: 0x00, disponible + Transfer ID)

    C1->>S: CHUNK_REQUEST (Transfer ID, chunk 0)
    S->>C2: CHUNK_REQUEST (Transfer ID, chunk 0)
    C2->>S: CHUNK_RESPONSE (Transfer ID, chunk 0, dades 0)
    S->>C1: CHUNK_RESPONSE (Transfer ID, chunk 0, dades 0)

    C1->>S: HASH_VERIFY (hash incorrecte)
    S->>C1: ERROR (Hash mismatch)

```

#### Resposta d'error estàndard

El següent flux mostra el funcionament d'una resposta d'error genèrica del protocol.

```mermaid
sequenceDiagram
    participant C as Client
    participant S as Servidor

    Note over C,S: Qualsevol operació que falli
    S->>C: ERROR (Error Code, Error Message)
    Note over C: Error Code pot ser:<br/>0x01 = Format invàlid<br/>0x02 = Fitxer no trobat<br/>0x03 = Accés denegat<br/>0x04 = Error del servidor<br/>0x05 = Checksum invàlid<br/>0x06 = Violació del protocol

```

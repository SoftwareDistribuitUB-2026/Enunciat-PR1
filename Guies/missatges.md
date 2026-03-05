# Glossari de missatges

Tots els missatges del protocol s'especifiquen a continuació

| Codi | Tipus de missatge | Direcció | Propòsit |
|------|------------------|----------|----------|
| 0x01 | CLIENT_REGISTER | Client → Servidor | Registrar el client |
| 0x02 | CLIENT_REGISTER_RESPONSE | Servidor → Client | Resposta de registre de client |
| 0x03 | FILE_ANNOUNCE | Client → Servidor | Anunciar els fitxers disponibles |
| 0x04 | SEARCH_REQUEST | Client → Servidor | Cercar fitxers |
| 0x05 | SEARCH_RESPONSE | Servidor → Client | Retornar els resultats de la cerca |
| 0x06 | DOWNLOAD_REQUEST | Client → Servidor | Sol·licitar la descàrrega d'un fitxer |
| 0x07 | DOWNLOAD_RESPONSE | Servidor → Client | Resposta a la sol·licitud de descàrrega |
| 0x08 | SERVER_FILE_REQUEST | Servidor → Client | El servidor sol·licita un fitxer al client origen |
| 0x09 | SERVER_FILE_RESPONSE | Client → Servidor | Resposta del client origen a la sol·licitud de fitxer |
| 0x0A | CHUNK_REQUEST | Bidireccional | Sol·licitar un fragment (Client→Servidor: client sol·licitant, Servidor→Client: el servidor el demana a l'origen) |
| 0x0B | CHUNK_RESPONSE | Bidireccional | Enviar dades del fragment (Client→Servidor: client origen, Servidor→Client: el servidor el reenvia al sol·licitant) |
| 0x0C | HASH_VERIFY | Bidireccional | Verificar el hash del fitxer |
| 0x0D | ERROR | Bidireccional | Missatge d'error |
| 0x0E | ACK | Bidireccional | Confirmació |



# Format dels missatges

Tots els missatges del protocol ARES segueixen un format estàndard consistent en dues parts principals:

## Estructura del missatge

Cada missatge està format per un **capçalera (header)** i una **càrrega útil (payload)**:

```
+------------------+------------------+
| Header           | Payload          |
| (fixa)           | (variable)       |
+------------------+------------------+
```

### Header

El capçalera conté la informació necessària per identificar i processar el missatge:

```
+------------------+
| Message Type     |
| (1 byte)         |
+------------------+
```

- **Message Type** (1 byte): Codi que identifica el tipus de missatge (vegeu la taula del glossari)


### Payload

La càrrega útil conté les dades específiques del missatge. La seva estructura i mida varien segons el tipus de missatge. Cada missatge té el seu propi format de payload, que es detalla a continuació a la secció d'especificació.




# Especificació

### Tipus de missatges i payloads

#### 1. CLIENT_REGISTER (0x01)
**Direcció:** Client → Servidor  
**Propòsit:** Registrar el client al servidor

**Payload:**
```
+------------------+------------------+
| Client ID Length | Client ID        |
| (4 bytes)        | (variable)       |
+------------------+------------------+
```

- **Client nickname Length** (4 bytes): Longitud de la cadena d'identificador del client, **numero de bytes**
- **Client nickname** (variable): Cadena d'identificador del client codificada en UTF-8

**Resposta:** El servidor envia el missatge CLIENT_REGISTER_RESPONSE

---

#### 2. CLIENT_REGISTER_RESPONSE (0x02)
**Direcció:** Servidor → Client 
**Propòsit:** Resposta al registrar el client al servidor

**Payload:**
```
+------------------+------------------+------------------+
| Status Code      | Client ID        | Chunk Size       |
| (1 byte)         | (2 bytes)        | (4 bytes)        |
+------------------+------------------+------------------+
```
- **Status Code** (1 byte):

0x00 = Èxit

0x01 = Error, no s'ha pogut registrar el client


- **Client ID** (2 bytes): L'ID del client generat pel servidor
- **Chunk size** (4 bytes): tamany dels fragments (chunks) a l'hora de transferir els fitxers



---

#### 3. FILE_ANNOUNCE (0x03)
**Direcció:** Client → Servidor  
**Propòsit:** Anunciar els fitxers disponibles al servidor

**Payload:**
```
+------------------+------------------+------------------+
| File Count       | File 1           | File 2           | 
| (4 bytes)        | (variable)      | (variable)        |
+------------------+------------------+------------------+

File x:
+------------------+------------------+------------------+------------------+
| Filename Length  | Filename         | File Size        | File Hash        |
| (4 bytes)        | (variable)       | (8 bytes)        | (32 bytes)       |
+------------------+------------------+------------------+------------------+
```

- **File Count** (4 bytes): Nombre de fitxers que s'anuncien
- Per a cada fitxer:
  - **Filename Length** (4 bytes): Longitud de la cadena del nom del fitxer en bytes
  - **Filename** (variable): Nom del fitxer codificat en UTF-8
  - **File Size** (8 bytes): Enter llarg, mida del fitxer en bytes
  - **File Hash** (32 bytes): Hash SHA-256 del fitxer

**Resposta:** El servidor envia `ACK` (0x0E) en cas d'èxit

---

#### 4. SEARCH_REQUEST (0x04)
**Direcció:** Client → Servidor  
**Propòsit:** Cercar fitxers per patró de nom de fitxer

**Payload:**
```
+------------------+------------------+
| Query Length     | Search Query     |
| (4 bytes)        | (variable)       |
+------------------+------------------+
```

- **Query Length** (4 bytes): nombre de bytes de la cadena de consulta de cerca
- **Search Query** (variable): Cadena de cerca codificada en UTF-8 (admet wildcards)

**Resposta:** El servidor envia `SEARCH_RESPONSE` (0x05)

---

#### 5. SEARCH_RESPONSE (0x05)
**Direcció:** Servidor → Client  
**Propòsit:** Retornar els resultats de la cerca amb informació dels clients

**Payload:**
```
+------------------+------------------+------------------+
| Result Count     | File 1           | File 2           | 
| (4 bytes)        | (variable)       | (variable)       |
+------------------+------------------+------------------+

Cada entrada de fitxer:
+------------------+------------------+------------------+------------------+
| Filename Length  | Filename         | File Size        | File Hash        |
| (4 bytes)        | (variable)       | (8 bytes)        | (32 bytes)       |
+------------------+------------------+------------------+------------------+
+------------------+------------------+------------------+
| Client Count     | Client ID 1      | Client ID 2      |
| (4 bytes)        | (2 bytes)        | (2 bytes)        |
+------------------+------------------+------------------+
```

- **Result Count** (4 bytes): Nombre de fitxers que coincideixen
- Per a cada fitxer:
  - **Filename Length** (4 bytes): Longitud en bytes de la cadena del nom del fitxer
  - **Filename** (variable): Nom del fitxer codificat en UTF-8
  - **File Size** (8 bytes): Enter llarg, mida del fitxer en bytes
  - **File Hash** (32 bytes): Hash SHA-256 del fitxer
  - **Client Count** (4 bytes): Nombre de clients que tenen aquest fitxer
  - Per a cada client:
    - **Client ID** (2 bytes): Identificador del client

---

#### 6. DOWNLOAD_REQUEST (0x06)
**Direcció:** Client → Servidor  
**Propòsit:** Sol·licitar la descàrrega d'un fitxer d'un altre client

**Payload:**
```
+------------------+------------------+------------------+
| Filename Length  | Filename         | Source Client ID |
| (4 bytes)        | (variable)       | (2 bytes)        |
+------------------+------------------+------------------+
```

- **Filename Length** (4 bytes): Longitud en bytes de la cadena del nom del fitxer
- **Filename** (variable): Nom del fitxer codificat en UTF-8 a descarregar
- **Source Client ID** (2 bytes): Identificador del client codificat (0 = qualsevol client disponible)

**Resposta:** El servidor envia `DOWNLOAD_RESPONSE` (0x07)

---

#### 7. DOWNLOAD_RESPONSE (0x07)
**Direcció:** Servidor → Client  
**Propòsit:** Resposta a la sol·licitud de descàrrega

**Payload:**
```
+------------------+------------------+------------------+------------------+
| Status Code      | Filename Length  | Filename         | File Size        |
| (1 byte)         | (4 bytes)        | (variable)       | (8 bytes)        |
+------------------+------------------+------------------+------------------+
+------------------+------------------+
| Source Client ID | Transfer ID      |
| (2 bytes)        | (4 bytes)        |
+------------------+------------------+
```

- **Status Code** (1 byte): 
  - `0x00` = Fitxer trobat, transferència iniciada (el servidor sol·licitarà al client origen)
  - `0x01` = Fitxer no trobat
  - `0x02` = Client origen no disponible
- **Filename Length** (4 bytes): Longitud en bytes del nom del fitxer (0 si el fitxer no es troba)
- **Filename** (variable): Nom del fitxer codificat en UTF-8 (buit si el fitxer no es troba)
- **File Size** (8 bytes): Enter llarg, mida del fitxer en bytes (0 si el fitxer no es troba)
- **Source Client ID** (variable): Identificador del client codificat amb dos bytes que proporciona el fitxer (buit si Status Code != 0x00)
- **Transfer ID** (4 byte): Identificador de la transferencia del fitxer

Si Status Code és `0x00`, el servidor enviarà `SERVER_FILE_REQUEST` al client origen per verificar que el fitxer està disponible.

---

#### 8. SERVER_FILE_REQUEST (0x08)
**Direcció:** Servidor → Client (Client origen)  
**Propòsit:** El servidor sol·licita un fitxer a un client en nom d'un altre client. Inclou el Transfer ID ja generat perquè els CHUNK_REQUEST posteriors puguin identificar la transferència.

**Payload:**
```
+------------------+------------------+------------------+------------------+
| Filename Length  | Filename         | Transfer ID      | Requesting Client ID |
| (4 bytes)        | (variable)       | (4 bytes)        | (2 bytes)        |
+------------------+------------------+------------------+------------------+
```

- **Filename Length** (4 bytes): Longitud en bytes de la cadena del nom del fitxer
- **Filename** (variable): Nom del fitxer codificat en UTF-8 a enviar
- **Transfer ID** (4 bytes): Identificador de la transferència ja generat pel servidor
- **Requesting Client ID** (2 bytes): Identificador del client que ha sol·licitat el fitxer

**Resposta:** El client origen envia `SERVER_FILE_RESPONSE` (0x09)

---

#### 9. SERVER_FILE_RESPONSE (0x09)
**Direcció:** Client (Origen) → Servidor  
**Propòsit:** El client origen respon a la sol·licitud de fitxer del servidor, confirmant si el fitxer està disponible i retornant el Transfer ID per consistència.

**Payload:**
```
+------------------+------------------+------------------+------------------+------------------+
| Status Code      | Transfer ID      | Filename Length  | Filename         | File Size        |
| (1 byte)         | (4 bytes)        | (4 bytes)        | (variable)       | (8 bytes)        |
+------------------+------------------+------------------+------------------+------------------+
+------------------+
| File Hash        |
| (32 bytes)       |
+------------------+
```

- **Status Code** (1 byte):
  - `0x00` = Fitxer llest, s'enviaran fragments
  - `0x01` = Fitxer no trobat o acces denegat
  - `0x02` = Client origen no disponible
- **Transfer ID** (4 bytes): Identificador de la transferència (retornat per confirmació)
- **Filename Length** (4 bytes): Longitud en bytes del nom del fitxer (0 si Status Code != 0x00)
- **Filename** (variable): Nom del fitxer codificat en UTF-8 (buit si Status Code != 0x00)
- **File Size** (8 bytes): Enter llarg, mida del fitxer en bytes (0 si Status Code != 0x00)
- **File Hash** (32 bytes): Hash SHA-256 del fitxer (tots zeros si Status Code != 0x00)

Si Status Code és `0x00`, el client origen ha d'esperar missatges `CHUNK_REQUEST` del servidor amb el mateix Transfer ID.

---

#### 10. CHUNK_REQUEST (0x0A)
**Direcció:** Bidireccional  
**Propòsit:** Sol·licitar un fragment específic d'un fitxer

**Ús:**
- **Client → Servidor** (Client sol·licitant): El client sol·licitant demana un fragment al servidor
- **Servidor → Client** (Client origen): El servidor demana un fragment al client origen

**Payload:**
```
+------------------+------------------+
| Transfer ID      | Chunk Number     |
| (4 bytes)        | (4 bytes)        |
+------------------+------------------+
```

- **Transfer ID** (4 bytes): Identificador de la transferència (el mateix que s'ha enviat a SERVER_FILE_REQUEST)
- **Chunk Number** (4 bytes): Índex del fragment basat en zero a sol·licitar

**Resposta:** 
- Si Client → Servidor: El servidor envia `CHUNK_RESPONSE` (0x0B) amb les dades del fragment (retransmès des del client origen)
- Si Servidor → Client: El client origen envia `CHUNK_RESPONSE` (0x0B) amb les dades del fragment

---

#### 11. CHUNK_RESPONSE (0x0B)
**Direcció:** Bidireccional  
**Propòsit:** Enviar dades del fragment

**Ús:**
- **Client → Servidor** (Client origen): El client origen envia el fragment al servidor
- **Servidor → Client** (Client sol·licitant): El servidor reenvia el fragment al client sol·licitant

**Payload:**
```
+------------------+------------------+------------------+------------------+
| Transfer ID      | Chunk Number     | Chunk Size       | Chunk Data       |
| (4 bytes)        | (4 bytes)        | (4 bytes)        | (variable)       |
+------------------+------------------+------------------+------------------+

```

- **Transfer ID** (4 bytes): ID de la transferencia
- **Chunk Number** (4 bytes): Índex del fragment basat en zero
- **Chunk Size** (4 bytes): Nombre de bytes en aquest fragment (0 indica final de fitxer o error)
- **Chunk Data** (variable): Bytes de dades del fitxer en brut (buit si Chunk Size és 0)

**Càlcul de la mida del fragment:**
- Fragments normals: `min(CHUNK_SIZE, remaining_bytes)`
- Últim fragment: `remaining_bytes` (pot ser menor que CHUNK_SIZE)
- Error/Final: `0`

**Flux:**
1. El client sol·licitant envia `CHUNK_REQUEST` al servidor
2. El servidor envia `CHUNK_REQUEST` al client origen
3. El client origen envia `CHUNK_RESPONSE` al servidor
4. El servidor reenvia `CHUNK_RESPONSE` al client sol·licitant

---

#### 12. HASH_VERIFY (0x0C)
**Direcció:** Client → Servidor o Servidor → Client  
**Propòsit:** Verificar la integritat del fitxer utilitzant hash

**Payload:**
```
+------------------+
| File Hash        |
| (32 bytes)       |
+------------------+
```

- **File Hash** (32 bytes): Hash SHA-256 amb el qual verificar

**Resposta:** El servidor/client envia `ACK` (0x0E) si el hash coincideix, `ERROR` (0x0D) si no coincideix

---

#### 13. ERROR (0x0D)
**Direcció:** Bidireccional  
**Propòsit:** Indicar una condició d'error

**Payload:**
```
+------------------+------------------+------------------+
| Error Code       | Message Length   | Error Message    |
| (1 byte)         | (4 bytes)        | (variable)       |
+------------------+------------------+------------------+
```

- **Error Code** (1 byte):
  - `0x01` = Format de missatge invàlid
  - `0x02` = Fitxer no trobat
  - `0x03` = Accés denegat
  - `0x04` = Error del servidor
  - `0x05` = Checksum invàlid
  - `0x06` = Violació del protocol
- **Message Length** (4 bytes): Longitud en bytes de la cadena del missatge d'error
- **Error Message** (variable): Descripció de l'error codificada en UTF-8

---

#### 14. ACK (0x0E)
**Direcció:** Bidireccional  
**Propòsit:** Confirmar una operació exitosa

**Payload:**
```
+------------------+
| (Empty)           |
+------------------+
```


**Resposta:** El servidor envia `ACK` (0x0E) en cas d'èxit, `ERROR` (0x0D) en cas de fallada

---

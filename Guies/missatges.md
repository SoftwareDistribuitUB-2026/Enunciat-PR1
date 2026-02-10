# Glossari de missatges

Tots els missatges del protocol s'especifiquen a continuació

| Codi | Tipus de missatge | Direcció | Propòsit |
|------|------------------|----------|----------|
| 0x01 | CLIENT_REGISTER | Client → Servidor | Registrar el client |
| 0x02 | FILE_ANNOUNCE | Client → Servidor | Anunciar els fitxers disponibles |
| 0x03 | SEARCH_REQUEST | Client → Servidor | Cercar fitxers |
| 0x04 | SEARCH_RESPONSE | Servidor → Client | Retornar els resultats de la cerca |
| 0x05 | DOWNLOAD_REQUEST | Client → Servidor | Sol·licitar la descàrrega d’un fitxer |
| 0x06 | DOWNLOAD_RESPONSE | Servidor → Client | Resposta a la sol·licitud de descàrrega |
| 0x0A | CHUNK_REQUEST | Bidireccional | Sol·licitar un fragment (Client→Servidor: client sol·licitant, Servidor→Client: el servidor el demana a l’origen) |
| 0x0B | CHUNK_RESPONSE | Bidireccional | Enviar dades del fragment (Client→Servidor: client origen, Servidor→Client: el servidor el reenvia al sol·licitant) |
| 0x0C | HASH_VERIFY | Bidireccional | Verificar el hash del fitxer |
| 0x0D | ERROR | Bidireccional | Missatge d’error |
| 0x0E | ACK | Bidireccional | Confirmació |
| 0x0F | SERVER_FILE_REQUEST | Servidor → Client | El servidor sol·licita un fitxer al client |
| 0x10 | SERVER_FILE_RESPONSE | Client → Servidor | Resposta del client a la sol·licitud de fitxer del servidor |


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

- **Client ID Length** (4 bytes): Longitud de la cadena d'identificador del client
- **Client ID** (variable): Cadena d'identificador del client codificada en UTF-8

**Resposta:** El servidor envia `ACK` (0x0E) en cas d'èxit, `ERROR` (0x0D) en cas de fallada

---

#### 2. FILE_ANNOUNCE (0x02)
**Direcció:** Client → Servidor  
**Propòsit:** Anunciar els fitxers disponibles al servidor

**Payload:**
```
+------------------+------------------+------------------+
| File Count       | File 1           | File 2           | 
| (4 bytes)        | (variable)      | (variable)        |
+------------------+------------------+------------------+

Cada entrada de fitxer:
+------------------+------------------+------------------+------------------+
| Filename Length  | Filename         | File Size        | File Hash        |
| (4 bytes)        | (variable)       | (8 bytes)        | (32 bytes)       |
+------------------+------------------+------------------+------------------+
```

- **File Count** (4 bytes): Nombre de fitxers que s'anuncien
- Per a cada fitxer:
  - **Filename Length** (4 bytes): Longitud de la cadena del nom del fitxer
  - **Filename** (variable): Nom del fitxer codificat en UTF-8
  - **File Size** (8 bytes): Enter llarg, mida del fitxer en bytes
  - **File Hash** (32 bytes): Hash SHA-256 del fitxer

**Resposta:** El servidor envia `ACK` (0x0E) en cas d'èxit

---

#### 3. SEARCH_REQUEST (0x03)
**Direcció:** Client → Servidor  
**Propòsit:** Cercar fitxers per patró de nom de fitxer

**Payload:**
```
+------------------+------------------+
| Query Length     | Search Query     |
| (4 bytes)        | (variable)       |
+------------------+------------------+
```

- **Query Length** (4 bytes): Longitud de la cadena de consulta de cerca
- **Search Query** (variable): Cadena de cerca codificada en UTF-8 (admet comodins)

**Resposta:** El servidor envia `SEARCH_RESPONSE` (0x04)

---

#### 4. SEARCH_RESPONSE (0x04)
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
+------------------+------------------+
| Client Count     | Client IDs       |
| (4 bytes)        | (variable)       |
+------------------+------------------+

Per a cada ID de client:
+------------------+------------------+
| Client ID Length | Client ID        |
| (4 bytes)        | (variable)       |
+------------------+------------------+
```

- **Result Count** (4 bytes): Nombre de fitxers que coincideixen
- Per a cada fitxer:
  - **Filename Length** (4 bytes): Longitud de la cadena del nom del fitxer
  - **Filename** (variable): Nom del fitxer codificat en UTF-8
  - **File Size** (8 bytes): Enter llarg, mida del fitxer en bytes
  - **File Hash** (32 bytes): Hash SHA-256 del fitxer
  - **Client Count** (4 bytes): Nombre de clients que tenen aquest fitxer
  - Per a cada client:
    - **Client ID Length** (4 bytes): Longitud de la cadena d'identificador del client
    - **Client ID** (variable): Identificador del client codificat en UTF-8

---

#### 5. DOWNLOAD_REQUEST (0x05)
**Direcció:** Client → Servidor  
**Propòsit:** Sol·licitar la descàrrega d'un fitxer d'un altre client

**Payload:**
```
+------------------+------------------+------------------+------------------+
| Filename Length  | Filename         | Source Client    | Source Client ID |
| (4 bytes)        | (variable)       | Length (4 bytes) | (variable)       |
+------------------+------------------+------------------+------------------+
```

- **Filename Length** (4 bytes): Longitud de la cadena del nom del fitxer
- **Filename** (variable): Nom del fitxer codificat en UTF-8 a descarregar
- **Source Client Length** (4 bytes): Longitud de l'identificador del client origen (0 = qualsevol client disponible)
- **Source Client ID** (variable): Identificador del client codificat en UTF-8 que té el fitxer (buit si Source Client Length és 0)

**Nota:** Si Source Client Length és 0, el servidor seleccionarà automàticament un client disponible.

**Resposta:** El servidor envia `DOWNLOAD_RESPONSE` (0x06)

---

#### 6. DOWNLOAD_RESPONSE (0x06)
**Direcció:** Servidor → Client  
**Propòsit:** Resposta a la sol·licitud de descàrrega

**Payload:**
```
+------------------+------------------+------------------+------------------+
| Status Code      | Filename Length  | Filename         | File Size        |
| (1 byte)         | (4 bytes)        | (variable)       | (8 bytes)        |
+------------------+------------------+------------------+------------------+
+------------------+------------------+------------------+
| File Hash        | Source Client    | Source Client ID |
| (32 bytes)       | Length (4 bytes) | (variable)       |
+------------------+------------------+------------------+
```

- **Status Code** (1 byte): 
  - `0x00` = Fitxer trobat, transferència iniciada (el servidor sol·licitarà al client origen)
  - `0x01` = Fitxer no trobat
  - `0x02` = Client origen no disponible
  - `0x03` = Accés denegat
- **Filename Length** (4 bytes): Longitud del nom del fitxer (0 si el fitxer no es troba)
- **Filename** (variable): Nom del fitxer codificat en UTF-8 (buit si el fitxer no es troba)
- **File Size** (8 bytes): Enter llarg, mida del fitxer en bytes (0 si el fitxer no es troba)
- **File Hash** (32 bytes): Hash SHA-256 del fitxer (tots zeros si el fitxer no es troba)
- **Source Client Length** (4 bytes): Longitud de l'identificador del client origen (0 si Status Code != 0x00)
- **Source Client ID** (variable): Identificador del client codificat en UTF-8 que proporciona el fitxer (buit si Status Code != 0x00)

Si Status Code és `0x00`, el client ha d'esperar missatges `CHUNK_RESPONSE` del servidor (el servidor retransmetrà els fragments del client origen).

---

#### 7. FILE_TRANSFER (0x09)
**Direcció:** Servidor → Client  
**Propòsit:** Transferir un fragment de dades del fitxer

**Payload:**
```
+------------------+------------------+------------------+
| Chunk Number     | Chunk Size       | Chunk Data       |
| (4 bytes)        | (4 bytes)        | (variable)       |
+------------------+------------------+------------------+
```

- **Chunk Number** (4 bytes): Índex del fragment basat en zero
- **Chunk Size** (4 bytes): Nombre de bytes en aquest fragment (normalment CHUNK_SIZE, excepte l'últim fragment)
- **Chunk Data** (variable): Bytes de dades del fitxer en brut

**Nota:** Aquest tipus de missatge pot estar en desús a favor del patró CHUNK_REQUEST/CHUNK_RESPONSE.

---

#### 8. CHUNK_REQUEST (0x0A)
**Direcció:** Bidireccional  
**Propòsit:** Sol·licitar un fragment específic d'un fitxer

**Ús:**
- **Client → Servidor** (Client sol·licitant): El client sol·licitant demana un fragment al servidor
- **Servidor → Client** (Client origen): El servidor demana un fragment al client origen

**Payload:**
```
+------------------+
| Chunk Number     |
| (4 bytes)        |
+------------------+
```

- **Chunk Number** (4 bytes): Índex del fragment basat en zero a sol·licitar

**Resposta:** 
- Si Client → Servidor: El servidor envia `CHUNK_RESPONSE` (0x0B) amb les dades del fragment (retransmès des del client origen)
- Si Servidor → Client: El client origen envia `CHUNK_RESPONSE` (0x0B) amb les dades del fragment

---

#### 8a. SERVER_FILE_REQUEST (0x0F)
**Direcció:** Servidor → Client (Client origen)  
**Propòsit:** El servidor sol·licita un fitxer a un client en nom d'un altre client

**Payload:**
```
+------------------+------------------+------------------+------------------+
| Filename Length  | Filename         | Requesting Client| Requesting ID    |
| (4 bytes)        | (variable)       | Length (4 bytes) | (variable)       |
+------------------+------------------+------------------+------------------+
```

- **Filename Length** (4 bytes): Longitud de la cadena del nom del fitxer
- **Filename** (variable): Nom del fitxer codificat en UTF-8 a enviar
- **Requesting Client Length** (4 bytes): Longitud de l'identificador del client sol·licitant
- **Requesting Client ID** (variable): Identificador del client codificat en UTF-8 que ha sol·licitat el fitxer

**Resposta:** El client origen envia `SERVER_FILE_RESPONSE` (0x10)

---

#### 8b. SERVER_FILE_RESPONSE (0x10)
**Direcció:** Client (Origen) → Servidor  
**Propòsit:** El client origen respon a la sol·licitud de fitxer del servidor

**Payload:**
```
+------------------+------------------+------------------+------------------+
| Status Code      | Filename Length  | Filename         | File Size        |
| (1 byte)         | (4 bytes)        | (variable)       | (8 bytes)        |
+------------------+------------------+------------------+------------------+
+------------------+
| File Hash        |
| (32 bytes)       |
+------------------+
```

- **Status Code** (1 byte):
  - `0x00` = Fitxer llest, s'enviaran fragments
  - `0x01` = Fitxer no trobat
  - `0x02` = Accés al fitxer denegat
- **Filename Length** (4 bytes): Longitud del nom del fitxer (0 si Status Code != 0x00)
- **Filename** (variable): Nom del fitxer codificat en UTF-8 (buit si Status Code != 0x00)
- **File Size** (8 bytes): Enter llarg, mida del fitxer en bytes (0 si Status Code != 0x00)
- **File Hash** (32 bytes): Hash SHA-256 del fitxer (tots zeros si Status Code != 0x00)

Si Status Code és `0x00`, el client origen ha d'esperar missatges `CHUNK_REQUEST` del servidor.

---

#### 9. CHUNK_RESPONSE (0x0B)
**Direcció:** Bidireccional  
**Propòsit:** Enviar dades del fragment

**Ús:**
- **Client → Servidor** (Client origen): El client origen envia el fragment al servidor
- **Servidor → Client** (Client sol·licitant): El servidor reenvia el fragment al client sol·licitant

**Payload:**
```
+------------------+------------------+------------------+
| Chunk Number     | Chunk Size       | Chunk Data       |
| (4 bytes)        | (4 bytes)        | (variable)       |
+------------------+------------------+------------------+
```

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

#### 10. HASH_VERIFY (0x0C)
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

#### 11. ERROR (0x0D)
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
- **Message Length** (4 bytes): Longitud de la cadena del missatge d'error
- **Error Message** (variable): Descripció de l'error codificada en UTF-8

---

#### 12. ACK (0x0E)
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

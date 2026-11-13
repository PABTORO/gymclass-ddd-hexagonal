El caso de uso **Book
Class** permite a un socio reservar una clase dirigida. El siguiente diagrama
ilustra el flujo de secuencia de extremo a extremo desde el cliente HTTP hasta
la base de datos relacional H2:
```mermaid
sequenceDiagram
    autonumber
    actor Cliente as Cliente HTTP (Postman / Thunder Client)
    participant Controller as GymClassController (Adaptador Entrada)
    participant InputPort as GymClassInputPort (Puerto Entrada)
    participant AppService as GymClassApplicationService (Servicio Aplicación)
    participant MemberPort as MemberRepositoryPort (Puerto Salida)
    participant SessionPort as ClassSessionRepositoryPort (Puerto Salida)
    participant Policy as BookingPolicy (Servicio Dominio)
    participant Member as Member (Agregado Dominio)
    participant Session as ClassSession (Raíz de Agregado)
    participant Adapter as ClassSessionRepositoryAdapter (Adaptador Salida)
    participant H2 as Base de Datos H2

    Cliente->>Controller: POST /api/sessions/{sessionId}/bookings/{memberId}
    Controller->>InputPort: bookClass(sessionId, memberId)
    InputPort->>AppService: bookClass(sessionId, memberId)
    
    AppService->>MemberPort: findById(memberId)
    MemberPort-->>AppService: Agregado Member
    
    AppService->>SessionPort: findById(sessionId)
    SessionPort-->>AppService: Agregado ClassSession
    
    AppService->>Policy: ensureEligible(member, session)
    Policy->>Member: canAttend(session.type)
    Member-->>Policy: boolean (true)
    Policy-->>AppService: Confirmación Elegibilidad
    
    AppService->>Session: book(member)
    Session->>Session: Valida capacidad e invariantes, crea Booking
    Session-->>AppService: Entidad Booking creada
    
    AppService->>SessionPort: save(session)
    SessionPort->>Adapter: save(session)
    Adapter->>Adapter: Mapea ClassSession a ClassSessionJpaEntity
    Adapter->>H2: save(sessionJpaEntity)
    H2-->>Adapter: Entidad Persistida
    Adapter-->>SessionPort: ClassSession actualizada
    SessionPort-->>AppService: ClassSession actualizada
    
    AppService-->>InputPort: DTO BookingResponse
    InputPort-->>Controller: DTO BookingResponse
    Controller-->>Cliente: 201 Created (JSON BookingResponse)
```

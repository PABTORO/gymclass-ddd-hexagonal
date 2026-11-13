```mermaid
graph TD
    subgraph DrivingAdapters["Adaptadores de Entrada (Driving / Primarios)"]
        RESTController["GymClassController<br/>(adapter/in/web)"]
    end

    subgraph HexagonCore["Núcleo de la Aplicación (Application & Domain)"]
        subgraph PortsIn["Puertos de Entrada"]
            InputPort["GymClassInputPort<br/>(application/port/in)"]
        end

        subgraph ApplicationLayer["Capa de Aplicación"]
            AppService["GymClassApplicationService<br/>(application/service)"]
        end

        subgraph DomainLayer["Capa de Dominio (Pure Java)"]
            DomainService["BookingPolicy<br/>(domain/service)"]
            AggMember["Member (Aggregate Root)<br/>(domain/model)"]
            AggClassSession["ClassSession (Aggregate Root)<br/>(domain/model)"]
            EntityBooking["Booking (Entity)<br/>(domain/model)"]
            ValueObjects["Value Objects<br/>(MemberId, SessionId, ClassSchedule...)"]
        end

        subgraph PortsOut["Puertos de Salida"]
            MemberRepoPort["MemberRepositoryPort<br/>(application/port/out)"]
            ClassSessionRepoPort["ClassSessionRepositoryPort<br/>(application/port/out)"]
        end
    end

    subgraph DrivenAdapters["Adaptadores de Salida (Driven / Secundarios)"]
        MemberRepoAdapter["MemberRepositoryAdapter<br/>(adapter/out/persistence)"]
        ClassSessionRepoAdapter["ClassSessionRepositoryAdapter<br/>(adapter/out/persistence)"]
        SpringDataMember["SpringDataMemberRepository"]
        SpringDataClassSession["SpringDataClassSessionRepository"]
        H2DB[("Base de Datos H2<br/>(jdbc:h2:mem:gymclassdb)")]
    end

    RESTController -->|1. Petición HTTP| InputPort
    AppService -.->|Implementa| InputPort
    AppService -->|2. Aplica regla| DomainService
    AppService -->|3. Opera sobre| AggMember
    AppService -->|4. Opera sobre| AggClassSession
    AggClassSession -->|Contiene| EntityBooking
    DomainService -->|Valida elegibilidad| AggMember
    DomainService -->|Valida elegibilidad| AggClassSession

    AppService -->|5. Carga / Guarda vía| MemberRepoPort
    AppService -->|6. Carga / Guarda vía| ClassSessionRepoPort

    MemberRepoAdapter -.->|Implementa| MemberRepoPort
    ClassSessionRepoAdapter -.->|Implementa| ClassSessionRepoPort

    MemberRepoAdapter -->|Utiliza| SpringDataMember
    ClassSessionRepoAdapter -->|Utiliza| SpringDataClassSession
    SpringDataMember -->|Persiste en| H2DB
    SpringDataClassSession -->|Persiste en| H2DB

---

## 🛠️ **2. Diagrama de Secuencia Caso de Uso `Book Class` (Código .md 100% Verificado)**

Este diagrama también ha sido revisado y cumple estrictamente con el estándar `sequenceDiagram`:

```markdown
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

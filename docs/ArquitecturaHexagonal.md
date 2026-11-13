````mermaid
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
    AggClassSession *-- EntityBooking
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

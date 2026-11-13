```mermaid
graph TD
    subgraph DrivingAdapters["Adaptadores de Entrada (Driving / Primarios)"]
        RESTController["GymClassController\n(adapter/in/web)"]
    end

    subgraph HexagonCore["Núcleo de la Aplicación (Application & Domain)"]
        subgraph PortsIn["Puertos de Entrada"]
            InputPort["GymClassInputPort\n(application/port/in)"]
        end

        subgraph ApplicationLayer["Capa de Aplicación"]
            AppService["GymClassApplicationService\n(application/service)"]
        end

        subgraph DomainLayer["Capa de Dominio (Pure Java)"]
            DomainService["BookingPolicy\n(domain/service)"]
            AggMember["Member (Aggregate Root)\n(domain/model)"]
            AggClassSession["ClassSession (Aggregate Root)\n(domain/model)"]
            EntityBooking["Booking (Entity)\n(domain/model)"]
            ValueObjects["Value Objects\n(MemberId, SessionId, ClassSchedule...)"]
        end

        subgraph PortsOut["Puertos de Salida"]
            MemberRepoPort["MemberRepositoryPort\n(application/port/out)"]
            ClassSessionRepoPort["ClassSessionRepositoryPort\n(application/port/out)"]
        end
    end

    subgraph DrivenAdapters["Adaptadores de Salida (Driven / Secundarios)"]
        MemberRepoAdapter["MemberRepositoryAdapter\n(adapter/out/persistence)"]
        ClassSessionRepoAdapter["ClassSessionRepositoryAdapter\n(adapter/out/persistence)"]
        SpringDataMember["SpringDataMemberRepository"]
        SpringDataClassSession["SpringDataClassSessionRepository"]
        H2DB[("Base de Datos H2\n(jdbc:h2:mem:gymclassdb)")]
    end

    RESTController -->|1. Petición HTTP| InputPort
    InputPort <|.. Implements | AppService
    AppService -->|2. Aplica regla| DomainService
    AppService -->|3. Opera sobre| AggMember
    AppService -->|4. Opera sobre| AggClassSession
    AggClassSession *-- EntityBooking
    DomainService -->|Valida elegibilidad| AggMember
    DomainService -->|Valida elegibilidad| AggClassSession

    AppService -->|5. Carga / Guarda vía| MemberRepoPort
    AppService -->|6. Carga / Guarda vía| ClassSessionRepoPort

    MemberRepoPort <|.. Implements | MemberRepoAdapter
    ClassSessionRepoPort <|.. Implements | ClassSessionRepoAdapter

    MemberRepoAdapter -->|Utiliza| SpringDataMember
    ClassSessionRepoAdapter -->|Utiliza| SpringDataClassSession
    SpringDataMember -->|Persiste en| H2DB
    SpringDataClassSession -->|Persiste en| H2DB
```

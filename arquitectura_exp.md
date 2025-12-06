# Arquitectura Blockchain Medicalchain - Explicación por Capas


## CAPA 1: Hardware e Infraestructura

![Capa Hardware e Infraestructura](./imagenes_arquitectura/Capa%20Hardware%20Infraestructura.JPG)

En el caso Medicalchain se menciona la necesidad de expandir su infraestructura más allá de un simple sistema de EHR blockchain. La empresa debe soportar:
- **Procesamiento de IA/ML** con alto poder computacional (GPUs)
- **Almacenamiento masivo** de imágenes médicas (radiografías, TACs)
- **Seguridad física de claves criptográficas** ante amenazas cuánticas
- **Edge computing** para dispositivos IoT médicos y wearables

Esta capa fundamental proporciona los recursos físicos sobre los que opera toda la plataforma:

**Nodos Blockchain Distribuidos:**
- Múltiples **nodos validadores** operados por hospitales partners y Medicalchain Core
- Cada nodo mantiene una copia completa del ledger y valida transacciones
- Distribuidos geográficamente para resiliencia y cumplimiento de regulaciones de residencia de datos

**Infraestructura Computacional Híbrida:**
- **Servidores On-Premise** en hospitales: mantienen control total sobre datos sensibles de pacientes
- **Cloud Híbrido (AWS/Azure/GCP)**: escalabilidad elástica con Kubernetes para orquestar microservicios
- **GPUs en la nube**: entrenan modelos de computer vision y NLP
- **Edge Computing**: dispositivos médicos procesan datos localmente reduciendo latencia crítica

**Hardware Especializado de Seguridad:**
- **HSM (Hardware Security Module)**: dispositivos físicos certificados FIPS 140-2 que protegen claves maestras
- **TPM/Secure Enclaves (TEE)**: chips Intel SGX o ARM TrustZone que crean zonas aisladas para procesar datos sin exponerlos
- **Aceleradores Quantum-Safe**: hardware especializado para ejecutar algoritmos criptográficos post-cuánticos (Kyber, Dilithium)

**Almacenamiento Físico:**
- **SAN/NAS**: sistemas de alta disponibilidad con redundancia para datos clínicos críticos
- **Object Storage**: almacenamiento S3-compatible escalable para imágenes médicas de gran tamaño
- **IPFS (InterPlanetary File System)**: sistema de archivos distribuido descentralizado para backups inmutables

Finalmente, aclarar que los nodos blockchain se hospedan en esta infraestructura híbrida, conectándose a los sistemas de almacenamiento para persistir el ledger y a HSMs para operaciones criptográficas seguras.

---

## CAPA 2: Red Blockchain

![Capa Red Blockchain](./imagenes_arquitectura/Capa%20red%20blockchain.JPG)

El caso destaca la necesidad de **arquitectura dual-chain**: una optimizada para EHR (alta privacidad, baja latencia) y otra para supply chain farmacéutica (alto throughput, trazabilidad). Además, menciona la integración con investigadores que requieren datos anonimizados.

Esta capa gestiona la comunicación entre nodos y organiza la red blockchain en canales especializados:

**Protocolo de Red P2P:**
- **Gossip Protocol (Hyperledger Fabric)**: propaga bloques nuevos entre nodos de manera eficiente, similar a cómo se esparcen rumores
- **TLS 1.3 + mTLS**: cifrado bidireccional donde ambos extremos se autentican con certificados
- **gRPC**: protocolo de comunicación de alto rendimiento basado en HTTP/2 para llamadas entre servicios

**Canales Hyperledger:**
- **Canal EHR**: ledger privado dedicado exclusivamente a registros médicos de pacientes. Solo hospitales y clínicas participan
- **Canal Supply Chain**: ledger separado para rastrear medicamentos desde fabricación hasta dispensación. Participan farmacéuticas, distribuidores y hospitales
- **Canal Research**: ledger para compartir datos médicos anonimizados con instituciones de investigación

**Gestión de Identidad de Red:**
- **MSP (Membership Service Provider)**: define qué organizaciones pueden participar en cada canal y sus permisos
- **Fabric CA (Certificate Authority)**: emite certificados X.509 que identifican y autentican a nodos, usuarios y organizaciones

Cuando un hospital registra un EHR, el nodo envía la transacción por gRPC a otros nodos del Canal EHR. El Gossip Protocol propaga el bloque resultante a todos los miembros del canal usando mTLS para seguridad. La Fabric CA valida que cada participante tiene certificados válidos emitidos según las políticas del MSP.

---

## CAPA 3: Consenso

![Capa Consenso](./imagenes_arquitectura/Capa%20Consenso.JPG)

El caso no especifica el mecanismo de consenso explícitamente, pero Hyperledger Fabric (mencionado en el caso) utiliza Raft por defecto. La necesidad de procesamiento rápido de transacciones médicas críticas requiere consenso de baja latencia.

Esta capa garantiza que todos los nodos acuerden el mismo orden de transacciones sin conflictos:

**Raft Consensus:**
- Algoritmo **CFT (Crash Fault Tolerant)**: tolera fallos por caída de nodos pero no ataques bizantinos maliciosos
- **Elección de líder**: los nodos eligen un líder que coordina el ordenamiento de transacciones
- **Replicación de log**: el líder replica las transacciones a nodos seguidores que confirman recepción
- Más rápido que PBFT (Byzantine Fault Tolerant) pero requiere confianza entre organizaciones participantes

**Ordering Service:**
- Recibe transacciones validadas de múltiples peers
- Las ordena cronológicamente en bloques de tamaño configurable
- Distribuye bloques ordenados a todos los nodos del canal

**Endorsement Policy:**
- Define cuántas y qué organizaciones deben firmar (endosar) una transacción antes de considerarse válida
- Ejemplo: "2 de 3 hospitales deben aprobar cambios al EHR del paciente"

1. Un smart contract ejecuta una transacción (ej: actualizar EHR)
2. Múltiples peers la validan según la **Endorsement Policy**
3. Las transacciones validadas llegan al **Ordering Service**
4. **Raft Consensus** garantiza que todos los nodos acuerden el orden
5. Se crea un bloque y se distribuye a todos los nodos del canal

---
## CAPA 4: Datos

![Capa Datos](./imagenes_arquitectura/Capa%20Datos.JPG)

El caso enfatiza múltiples necesidades de datos:
- **Blockchain para EHR y Supply Chain**: ledger inmutable con audit trail
- **Smart contracts** para automatizar consentimientos, pagos y trazabilidad
- **Cifrado homomórfico**: procesar datos médicos con IA sin descifrarlos (caso menciona overhead 10-100x)
- **Criptografía post-cuántica**: preparación ante amenaza cuántica en 5-10 años

Esta capa gestiona el almacenamiento y cifrado de todos los datos del sistema:

**Ledger Blockchain:**
- **World State DB (CouchDB)**: base de datos NoSQL que almacena el estado actual de todos los activos (pacientes, medicamentos, tokens). Permite queries JSON complejas
- **Blockchain Log**: cadena inmutable de bloques que registra históricamente todas las transacciones. Funciona como "source of truth" para auditorías
- **Private Data Collections**: almacenamiento cifrado donde solo organizaciones autorizadas específicas acceden (ej: resultados de VIH solo para médico tratante)

**Smart Contracts (Chaincode):**
- **EHR Smart Contract**: código que ejecuta lógica de negocio para gestionar historiales médicos, consentimientos, accesos temporales
- **Supply Chain Contract**: registra cada paso del medicamento (fabricación, almacenamiento, temperatura, transporte, dispensación)
- **Financial Contract**: automatiza pagos a médicos, procesamiento de reclamaciones a aseguradoras, transacciones con MedToken
- **Research Contract**: gestiona compartir datos anonimizados con investigadores, distribuye incentivos en MedTokens a pacientes

**Bases de Datos Off-Chain:**
- **BD Clínica (PostgreSQL)**: almacena EHRs completos con cifrado TDE (Transparent Data Encryption). El blockchain solo guarda hashes y metadatos
- **BD Operacional (MongoDB)**: gestiona citas, usuarios, configuraciones del sistema
- **Data Lake (Hadoop/Spark)**: repositorio masivo de datos crudos para análisis estadístico y entrenamiento de modelos de IA
- **Time-Series DB (InfluxDB)**: optimizada para datos continuos de wearables y sensores IoT

**Cifrado Avanzado:**
- **Cifrado en Reposo (AES-256-GCM)**: todos los datos almacenados están cifrados
- **Cifrado en Tránsito (TLS 1.3)**: comunicaciones de red cifradas
- **Cifrado Homomórfico**: permite ejecutar algoritmos de IA sobre datos cifrados sin descifrarlos nunca. Crítico para cumplir GDPR/HIPAA al procesar en nube pública
- **Criptografía Ágil**: arquitectura que permite cambiar algoritmos criptográficos sin rediseñar el sistema. Esencial para migrar a algoritmos post-cuánticos (Kyber, Dilithium)

Cuando un médico actualiza un EHR:
1. El **EHR Service** invoca el **EHR Smart Contract** con los cambios
2. El smart contract valida permisos y consentimientos
3. Escribe el hash del EHR en el **World State DB** y el **Blockchain Log**
4. Los datos completos se cifran con **AES-256** y se almacenan en **BD Clínica**
5. Las claves de cifrado se gestionan en el **KMS** respaldado por **HSM**

---

## CAPA 5: Aplicación - Parte 1

![Capa de Aplicación Parte 1](./imagenes_arquitectura/Capa%20de%20aplicaci%C3%B3n%20parte%201.JPG)


El caso detalla las iniciativas estratégicas que Medicalchain debe implementar:
- **Servicios de dominio**: EHR, telemedicina, supply chain farmacéutica
- **IA/ML**: computer vision para radiología, NLP para notas clínicas, analítica predictiva
- **Federated Learning**: entrenar IA sin centralizar datos (cumple GDPR/HIPAA)

**Servicios de Dominio (Microservicios):**
- **EHR Service**: gestiona CRUD de registros médicos, invoca EHR chaincode, genera audit trails
- **Telemedicine Service**: videoconsultas, gestión de citas, prescripciones electrónicas
- **Patient Management**: administra perfiles, consentimientos GDPR/HIPAA, preferencias de privacidad
- **Supply Chain Service**: rastrea medicamentos, integra sensores IoT de temperatura en la cold chain
- **Financial Service**: procesa pagos, reclamaciones a aseguradoras, gestiona billeteras MedToken
- **Research Service**: anonimiza datos para investigación, gestiona marketplace de datos científicos

**Servicios de IA/ML:**
- **Computer Vision Service**: analiza imágenes médicas (rayos X, MRIs, TACs) detectando anomalías con 95%+ precisión
- **NLP Service**: extrae información estructurada de notas clínicas escritas en lenguaje natural, genera códigos ICD-10
- **Predictive Analytics**: modelos estadísticos que predicen riesgos de readmisión, interacciones medicamentosas, deterioro del paciente
- **Federated Learning Orchestrator**: coordina entrenamiento distribuido de modelos de IA en múltiples hospitales sin transferir datos sensibles. Cada hospital entrena localmente y solo comparte actualizaciones del modelo

Para analizar una radiografía con IA:
1. El médico sube la imagen al **EHR Service**
2. El servicio la envía cifrada con **cifrado homomórfico** al **Computer Vision Service**
3. El servicio IA procesa la imagen cifrada sin descifrarla (overhead 10-100x pero mantiene privacidad)
4. Alternativamente, puede invocar **AI Cloud Providers** (AWS SageMaker) con GPUs enviando datos cifrados
5. El resultado (ej: "posible neumonía detectada en lóbulo inferior derecho") se registra en el EHR
6. El **Federated Learning Orchestrator** usa este caso anonimizado para mejorar el modelo sin extraer datos

---

## CAPA 5: Aplicación - Parte 2

![Capa de Aplicación Parte 2](./imagenes_arquitectura/Capa%20de%20Aplicaci%C3%B3n%20parte%202.JPG)

El caso menciona tecnologías emergentes que Medicalchain debe evaluar:
- **AR/VR**: consultas inmersivas, planificación quirúrgica, terapia de dolor
- **Digital Twins**: réplicas virtuales de pacientes para simular tratamientos

**Servicios XR (Extended Reality):**
- **VR Consultation Service**: plataforma de telemedicina inmersiva donde paciente y médico interactúan en entorno virtual 3D. Aplicaciones en terapia de manejo de dolor y exposición para fobias
- **AR Surgery Planning Service**: superpone imágenes médicas (TAC/MRI) en el campo quirúrgico real usando headsets HoloLens. Permite ensayar cirugías complejas
- **XR Therapy Service**: aplicaciones de fisioterapia y rehabilitación donde pacientes realizan ejercicios en entornos gamificados

**Digital Twin Platform:**
- **Patient Digital Twin Engine**: crea réplica computacional del sistema fisiológico del paciente integrando genética, historial médico, datos de wearables. Simula progresión de enfermedades y respuesta a tratamientos
- **Asset Digital Twin**: réplica virtual de equipos médicos para mantenimiento predictivo
- **Simulation Engine**: ejecuta simulaciones de pruebas de fármacos sobre miles de digital twins antes de ensayos clínicos reales (reduce costos y riesgos)

**APIs y Gateway:**
- **API Gateway**: punto de entrada único con OAuth2, rate limiting, WAF (Web Application Firewall). Protege contra ataques DDoS, inyección SQL, XSS
- **GraphQL API**: permite a clientes solicitar exactamente los datos que necesitan en una sola query (más eficiente que REST)
- **WebSocket Gateway**: conexión bidireccional persistente para comunicación en tiempo real (videollamadas, monitoreo de signos vitales)

Para planificación quirúrgica con AR:
1. El cirujano solicita el **Digital Twin** del paciente desde **AR Surgery Planning Service**
2. El servicio consulta **Patient Digital Twin Engine** que lee datos de **BD Clínica**
3. El twin se carga con imágenes TAC/MRI del **Object Storage**
4. El cirujano usa headset HoloLens para visualizar anatomía superpuesta en 3D
5. Ensaya la cirugía virtualmente, ajustando el plan quirúrgico
6. Los datos de la planificación se registran en el EHR vía **EHR Service**

---

## CAPA 6: Seguridad y Privacidad (Zero Trust)

![Capa de Seguridad](./imagenes_arquitectura/Capa%20de%20Seguridad.JPG)

El caso enfatiza la necesidad crítica de **Zero Trust Architecture**:
- **Microsegmentación**: aislar cada microservicio
- **SIEM + UEBA + SOAR**: detección automatizada de amenazas y respuesta
- **Breach de 190 millones de registros en 2024**: urgencia de seguridad proactiva
- **Harvest now, decrypt later**: adversarios capturan datos cifrados hoy para descifrarlos cuando exista computación cuántica

**Identidad y Acceso:**
- **IAM/IdP**: gestión centralizada de identidades con SSO (Single Sign-On), MFA obligatorio (contraseña + huella + SMS), autenticación biométrica
- **ABAC Engine (Attribute-Based Access Control)**: evalúa contexto completo antes de otorgar acceso: rol del usuario, sensibilidad de datos, ubicación geográfica, hora del día, nivel de amenaza actual
- **JIT Access (Just-In-Time)**: provisiona permisos temporalmente solo cuando se necesitan específicamente. Ejemplo: médico obtiene acceso a EHR solo durante turno de guardia

**Gestión de Llaves y Secretos:**
- **KMS Centralizado**: gestiona ciclo de vida completo de llaves criptográficas (creación, rotación automática cada 90 días, revocación, auditoría)
- **HSM Distribuido**: dispositivos físicos certificados que protegen claves maestras en múltiples geografías
- **Secrets Management (Vault)**: bóveda que almacena y rota automáticamente contraseñas, API keys, tokens de DB sin exponerlas a desarrolladores

**Monitoreo y Respuesta:**
- **SIEM (Security Information and Event Management)**: agrega logs de todos los componentes, correlaciona eventos para detectar patrones de ataque (ej: 50 intentos fallidos de login en 5 minutos)
- **UEBA (User and Entity Behavior Analytics)**: IA que aprende el comportamiento normal de cada usuario/sistema y detecta anomalías (ej: médico accede 500 EHRs en 1 hora cuando su promedio es 20/día)
- **SOAR (Security Orchestration, Automation and Response)**: automatiza respuesta a incidentes sin intervención humana. Si UEBA detecta anomalía, SOAR bloquea automáticamente la cuenta, revoca tokens, notifica al SOC
- **Threat Intelligence Feeds**: flujos de información sobre amenazas globales actualizadas (framework MITRE ATT&CK)

**Seguridad de Red:**
- **Microsegmentación**: cada microservicio tiene su propio firewall virtual. Si un servicio se compromete, el atacante no puede moverse lateralmente
- **DDoS Protection**: CloudFlare/AWS Shield filtran tráfico malicioso masivo antes de llegar a la infraestructura
- **IDS/IPS**: monitorea tráfico de red en tiempo real, detecta patrones de ataque (port scanning, buffer overflow) y bloquea automáticamente

**Privacidad y Compliance:**
- **Anonymization Engine**: aplica técnicas como k-anonymity (generalizar datos hasta que sean indistinguibles entre k personas) y differential privacy (agregar ruido estadístico)
- **Consent Management**: registra consentimientos explícitos del paciente para cada uso de datos. Cumple GDPR (derecho al olvido) y HIPAA
- **Audit & Compliance Reporting**: genera reportes automáticos demostrando cumplimiento regulatorio. El blockchain proporciona audit trail inmutable
- **Data Retention Policies**: elimina automáticamente datos según requerimientos legales (GDPR: máximo 7 años) y preferencias del paciente

Cuando un usuario intenta acceder a un EHR:
1. **IAM/IdP** autentica con MFA (contraseña + biometría)
2. **ABAC Engine** evalúa: ¿es médico tratante? ¿está en horario laboral? ¿accede desde hospital? ¿nivel de amenaza bajo?
3. **JIT Access** provisiona permiso temporal de 8 horas
4. **API Gateway** registra el acceso en **SIEM**
5. **UEBA** compara con comportamiento histórico del usuario
6. Si todo es normal, el acceso se permite y se registra en el **Blockchain Log** (audit trail inmutable)
7. Si **UEBA** detecta anomalía, **SOAR** bloquea automáticamente y alerta al equipo de seguridad

---

## CAPA 7: Interfaces de Usuario y Sistemas Externos

![Capa Interfaz de Usuario y Externos](./imagenes_arquitectura/Capa%20interfaz%20de%20usuario%20y%20externos.JPG)

El caso menciona:
- **MyClinic.com**: app de telemedicina de Medicalchain
- **450,000 pacientes registrados y 2,300 proveedores**: requiere interfaces escalables
- **Integración con EHR legados** (Epic, Cerner): estándar HL7 FHIR
- **Developer platform**: APIs públicas para third-party apps

**Interfaces de Usuario:**
- **Mobile App MyClinic (iOS/Android)**: aplicación nativa para pacientes con acceso a historial médico, agendar citas, videoconsultas, gestionar consentimientos
- **Web Portal Médico (React/Vue SPA)**: aplicación web de página única para profesionales de salud con herramientas clínicas completas, prescripción electrónica, visualización de imágenes médicas
- **XR Client (Unity/Unreal Engine)**: aplicaciones inmersivas para dispositivos VR/AR como Meta Quest, HoloLens
- **Third-Party DApps**: ecosistema de desarrolladores externos que construyen aplicaciones integrándose vía APIs públicas de Medicalchain

**Sistemas Externos:**
- **Hospital EHR Legado (Epic, Cerner)**: sistemas antiguos de hospitales que deben integrarse. Usan estándar **HL7 FHIR** (Fast Healthcare Interoperability Resources) para intercambio de datos
- **Aseguradora**: compañías de seguros que procesan reclamaciones y reembolsos vía APIs B2B con mTLS
- **ERP Farmacéutica (SAP, Oracle)**: sistemas de fabricantes que controlan producción. Se integran para supply chain tracking
- **WMS (Warehouse Management System)**: software de distribuidores que controla inventario y movimiento de medicamentos
- **Customs/Regulatory Agencies**: entidades gubernamentales con acceso read-only limitado para supervisión regulatoria
- **AI Cloud Providers (AWS SageMaker, Azure ML)**: servicios externos con GPUs para entrenar y ejecutar modelos de IA complejos

Cuando un paciente agenda una cita:
1. El paciente usa **Mobile App MyClinic**
2. La app se autentica con el **API Gateway** usando OAuth2 + MFA
3. El gateway valida el token con **IAM/IdP**
4. Enruta la solicitud al **Telemedicine Service**
5. El servicio crea la cita en **BD Operacional**
6. Notifica al médico vía **WebSocket Gateway** en tiempo real
7. El evento se registra en **SIEM** para auditoría
8. Si el hospital usa **Epic (EHR Legado)**, la cita se sincroniza vía **HL7 FHIR**

---

## Flujos de Datos Integrados: Ejemplo Completo

### Caso de Uso: Diagnóstico Asistido por IA con Trazabilidad Completa

**Escenario:** Un paciente llega a urgencias con dolor torácico. El médico solicita rayos X y el sistema asiste en el diagnóstico.

**Flujo Completo:**

1. **Capa 7 (UI):** Médico solicita rayos X desde **Web Portal Médico**
2. **Capa 6 (Seguridad):** **API Gateway** autentica con **IAM/IdP**, **ABAC Engine** valida permisos contextuales
3. **Capa 5 (Aplicación):** **EHR Service** recibe la solicitud
4. **Capa 4 (Datos):** **EHR Smart Contract** registra la orden en el **Blockchain Log** (audit trail inmutable)
5. Técnico de radiología toma la imagen, se sube al **Object Storage** cifrada con **AES-256**
6. **Capa 5 (IA/ML):** **Computer Vision Service** analiza la imagen
   - Opción A: Procesa localmente con **cifrado homomórfico** (sin descifrar)
   - Opción B: Envía cifrada a **AI Cloud Provider** con GPUs para procesamiento más rápido
7. El servicio IA detecta "posible neumonía en lóbulo inferior derecho - confianza 94%"
8. **Capa 4:** Resultado se registra en **BD Clínica** y hash en **World State DB**
9. **Capa 6 (Monitoreo):** **SIEM** registra todos los accesos, **UEBA** valida comportamiento normal
10. **Capa 5 (Digital Twin):** **Patient Digital Twin Engine** se actualiza con el diagnóstico, simula progresión si no se trata
11. Médico prescribe antibióticos
12. **Capa 5 (Supply Chain):** **Supply Chain Service** invoca **Supply Chain Contract**
13. **Capa 4:** Se registra en el **Canal Supply Chain** del blockchain
14. **Capa 2 (Red):** **Gossip Protocol** propaga la transacción a farmacia del hospital
15. **Capa 3 (Consenso):** **Raft Consensus** garantiza que todos los nodos acuerden el registro
16. Farmacia dispensa medicamento, sensores IoT registran que se mantuvo en temperatura correcta
17. **Capa 1 (Hardware):** Datos de sensores **Edge Computing** se almacenan en **Time-Series DB**
18. **Capa 6 (Compliance):** **Audit & Compliance Reporting** genera reporte HIPAA automáticamente
19. **Capa 7 (Externos):** Si paciente tiene seguro, **Aseguradora** recibe reclamación vía **API Gateway** con mTLS

**Beneficios de la Arquitectura:**
- **Trazabilidad completa**: cada acción registrada inmutablemente en blockchain
- **Privacidad**: datos procesados con cifrado homomórfico sin exposición
- **Seguridad**: Zero Trust con autenticación continua y UEBA detectando anomalías
- **Inteligencia**: IA asiste en diagnóstico con 94% precisión
- **Preparación futura**: criptografía ágil lista para migrar a algoritmos post-cuánticos

---

## Conclusiones

Esta arquitectura de 7 capas aborda todos los desafíos estratégicos del caso Medicalchain y presentamos lo siguiente como conclusiones:

**Blockchain dual-chain**: canales separados para EHR (privacidad) y Supply Chain (throughput)  
**IA/ML integrada**: computer vision, NLP, predictive analytics con federated learning  
**Tecnologías emergentes**: XR para telemedicina inmersiva, Digital Twins para medicina personalizada  
**Seguridad robusta**: Zero Trust con SIEM+UEBA+SOAR, microsegmentación, cifrado homomórfico  
**Preparación cuántica**: criptografía ágil con algoritmos post-cuánticos (Kyber, Dilithium)  
**Cumplimiento regulatorio**: GDPR/HIPAA compliant con consent management y audit trails inmutables  
**Escalabilidad**: infraestructura híbrida cloud-edge con Kubernetes  
**Interoperabilidad**: integración HL7 FHIR con EHR legados, APIs públicas para desarrolladores  

La arquitectura posiciona a Medicalchain para competir efectivamente en el mercado de healthcare IT de $390 mil millones, diferenciándose mediante la combinación única de blockchain, IA y seguridad de nivel militar.


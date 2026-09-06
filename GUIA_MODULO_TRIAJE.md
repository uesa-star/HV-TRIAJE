# GUÍA DEL MÓDULO DE TRIAJE SGH — Hospital Ventanilla

**Proyecto:** Sistema Inteligente de Triaje (Emergencias SGH)
**Inicio:** 05/09/2026 · **Repo:** uesa-star/TRIAJE · **Despliegue:** Vercel (triaje-ten.vercel.app)
**Archivos:** `index.html` (módulo) · `GUIA_MODULO_TRIAJE.md` + `.pdf` (esta guía)

---

## 1. ¿Qué es y para quién es?

Módulo web para gestionar el **triaje de emergencia**: registra pacientes, clasifica prioridad clínica (ESI P-I a P-IV), controla la cola de espera en tiempo real, vigila eventos epidemiológicos y genera reportes operativos.

- **Institución:** Hospital de Ventanilla (salud pública) — servicio de Emergencia/Triaje.
- **Usuarios:** admisión/triaje, médicos de guardia, jefatura de emergencia, estadística, epidemiología.
- **Rubro adaptable:** clínicas privadas, centros y puestos de salud con urgencias.

## 2. Estructura del menú (abre colapsado, solo iconos)

| Orden | Ítem | Vista | Contenido |
|---|---|---|---|
| 1 | 📝 Ingreso de Pacientes | `ingreso` | Formulario completo (vista inicial). Botones: 💾 Guardar Ingreso + 🧹 Limpiar |
| 2 | 📋 Reporte de Triaje | `reporte` | 5 KPIs + tabla general, buscador, filtros, Excel |
| 3 | ⏳ Cola de Espera | `cola` | En Espera / En Atención / Cerrados, cronómetros en vivo |
| 4 | 📊 Dashboard Real-Time | `dashboard` | KPIs del turno, flujo por hora, distribución por prioridad |
| 5 | 📈 Analítica Operativa | `estadisticas` | 5 gráficos + KPIs (hora, prioridad, especialidad, área, día) |
| 6 | 🧪 Laboratorio | `lab` | Buscador (Triaje+LAB), ingreso LAB, pedidos → resultados, ficha PDF |

Encabezado: 🩺 SGH TRIAJE (marca neutra multi-institución) · título de vista · KPIs (espera promedio, críticos) · estado de conexión · Fecha Sistema + hora en vivo · 👤 Dr. Guardia Triaje + estado. Fecha y hora usan **hora local del Perú** (no UTC).

## 3. Flujo de trabajo y estados

1. **Ingreso** (vista Ingreso o ➕ Admitir en celular) → filiación + signos vitales → prioridad ESI automática. Si el documento ya ingresó **hoy**, sale popup de reingreso (Seguir / No continuar).
2. El paciente entra a la **Cola de Espera** ordenado por prioridad y hora.
3. **Iniciar** → `en_atencion` (médico + hora inicio). **Cerrar** → `atendido` (diagnóstico, área, medicación, vacuna, hora fin). **Reabrir** si hace falta.
4. **🚪 Retirar**: el paciente se va sin atención → estado `retirado` (queda registro). **↩ Reingresar** lo devuelve a la cola.
5. Todo visible en Reporte, Cola, Dashboard y Analítica. Los KPIs del Reporte responden a los filtros.

## 4. Formulario de triaje (campos y orden)

**Fila 1:** Tipo Doc * (DNI, C.E., C.P.P., Pasaporte, Cédula Ext., Otro) · N° Documento * · Apellidos y Nombres * (ancho) · —.
**Fila 2:** Edad * (número + lista Días/Meses/Años) · HC (vacía: **la asigna el servidor**) · Sexo * · Especialidad.
**Signos (los toma la enfermera):** PA sist/diast · FC · FR · SpO₂ · T° · Talla · Peso · Motivo *. **Sin glucosa** (es dato de laboratorio).
**Autocompletado al escribir el documento:** Nivel 1 base local (nombres, edad, sexo, HC) · Nivel 2 RENIEC por DNI (gancho listo, `RENIEC_CONFIG`, requiere convenio PIDE). **Validación:** DNI = 8 dígitos exactos.

## 5. Clasificación de prioridad (ESI automático)

`calculateTriagePriority()`: PA, FC, FR, SpO₂, T° (glucosa solo si viene de lab) → **critico (P-I) · alto (P-II) · moderado (P-III) · estable (P-IV)**, con alertas (HTA, hipotensión, taquicardia, taquipnea, SatO2 baja, hipoxia, fiebre, hiperglucemia).

## 6. Vigilancia epidemiológica automática (MINSA/CDC)

`evaluarVigilancia()`: **FEBRIL** (T° ≥ 38) · **SOBA** (sibilancias, asma, tiraje, espasmo) · **IRA** (FR > 20, Sat < 95 %, tos, disnea, gripe, neumonía…) · **EDA** (diarrea, vómitos, deshidratación). Mensaje rojo al guardar + distintivo 🦠 en tabla + sección en la ficha. **Etapa de vida:** Niño (0-11) · Adolescente (12-17) · Joven (18-29) · Adulto (30-59) · Adulto mayor (60+).

## 7. Excel (formato .xls normal, sin internet)

Botón 📥 Excel → `Reporte_Triaje_AAAA-MM-DD.xls` con encabezado azul. Columnas: Fecha, Hora, Paciente, Tipo_Doc, DNI, HC, Edad, Sexo, Especialidad, Estado, PA_Sist, PA_Diast, FC, FR, SpO2, Temp, Glucosa, Riesgo, **Etapa_Vida, Vig_EDA, Vig_IRA, Vig_SOBA, Vig_Febril** (SÍ/vacío, contables con CONTAR.SI), Medico, Diagnostico, Area_Destino, Medicacion, Vacuna.

## 8. Módulo Laboratorio (local; API futura `/api/lab`)

Vista `lab` (🧪): **buscador** por DNI/apellidos/HC en Triaje + LAB · **📥 Traer de Triaje a LAB** (copia filiación) · **➕ Nuevo en LAB** (ingreso rápido) · flujo en 2 pasos: **📋 Pedido** (examen solicitado) → **✏️ Ingresar** (carga valor) → **✅ Ingresado** · botón **🖨️ PDF** genera la ficha de resultados (_EL diálogo de impresión → Guardar como PDF_).

## 9. KPIs del Reporte (un solo color neutro, listo para modo oscuro)
📥 Ingresados · 👥 Etapa frecuente · 🕐 Hora pico · 🩺 Dx frecuente (agrupador: Respiratorio, Digestivo, Trauma, Cardiovascular, Neurológico, Febril…) · 🦠 Vigilancia top. Se recalculan con cada filtro.

## 10. Capa de datos y contrato API (para el inge)

`API_CONFIG`: `baseUrl: 'http://172.18.26.18:8000'`, timeout 3 s, sondeo 10 s.

| Método | Ruta | Uso |
|---|---|---|
| GET | `/api/triajes` | Lista completa (también chequeo de conexión) |
| POST | `/api/triajes` | Crea triaje (upsert por `id` de cliente) |
| PUT | `/api/triajes/{id}` | Actualiza (estado, cierre, retiro…) |

🟢 Conectado = sondeo + fusión cada 10 s. 🟡 Modo Local = resguardo `localStorage` (`sgh_triaje_v2`), sube pendientes al reconectar. **La HC la asigna el servidor** (viene vacía del formulario). **Nota de red:** Vercel es HTTPS; abrir por IP directo en el hospital o API con HTTPS.

## 11. Blindaje

Escape anti-XSS en todos los datos de paciente · clic derecho bloqueado en tabla y colas · aviso en consola (Ley 29733). Pendiente lado servidor: cabecera anti-clickjacking (Vercel/API).

## 12. Historial de modificaciones

1–15: base (persistencia, flujo, cola, analítica, API+modo local, header, menú, ingreso, tipo doc, vista inicial, usuario, anchos, guía).
16. Guía en PDF · 17. Excel normal .xls · 18. Vigilancia FEBRIL/SOBA/IRA/EDA + columna · 19. Hora + fecha local Perú · 20. Excel por ítems + etapa de vida · 21. Orden Etapa, EDA, IRA, SOBA, Febril · 22. Sin glucosa en ingreso · 23. Botón arriba + compacto · 24. DNI primero + RENIEC + HC auto · 25. Fila Edad→HC→Sexo→Especialidad · 26. Edad junto a Nombres · 27. Retirar/Reingresar · 28. Edad número + lista · 29. Edad abajo junto a HC · 30. HC vacía (la asigna servidor) · 31. Autocompletado base local + RENIEC · 32. Botón Limpiar · 33–34. Orden y grupo de botones · 35. KPIs del Reporte · 36. KPIs un color · 37. Dx frecuente · 38. Menú inicia cerrado · 39. Blindaje · 40–42. Alerta reingreso mismo día (popup al escribir y al guardar) · 43. Guía siempre junto a cada subida · 44. Marca neutra SGH TRIAJE (multi-institución: hospital, LAB, más módulos) · 45. Módulo Laboratorio (buscador, ingreso LAB, resultados) · 46. LAB en 2 pasos (Pedido → Ingresado) · 47. Ficha LAB en PDF.

## 13. Pendientes para aprobación del inge

- [ ] Almacenamiento final (API hospital / Supabase) y retiro de datos demo.
- [ ] Impresión real del ticket de triaje.
- [ ] Validar rangos ESI y grupos Dx con el área médica/epidemiología.
- [ ] RENIEC vía PIDE (convenio) + HC correlativa desde el servidor.
- [ ] Usuarios por rol + auditoría (Ley 29733) + cabecera anti-clickjacking.
- [ ] Botón modo claro/oscuro.
- [ ] Pruebas con datos reales en la red del hospital.

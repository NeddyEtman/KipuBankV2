# KipuBank V2 - Protocolo de Bóveda DeFi (Multi-Asset & Oráculos)

![License](https://img.shields.io/badge/License-MIT-blue.svg)
![Solidity](https://img.shields.io/badge/Solidity-%5E0.8.26-363636)
![Network](https://img.shields.io/badge/Network-Sepolia_Testnet-orange)
![Status](https://img.shields.io/badge/Verified-0x20D8...108F-success)

**KipuBankV2** es una evolución de una bóveda de almacenamiento simple hacia un protocolo financiero robusto. Permite la custodia segura de ETH y tokens ERC-20, utilizando precios en tiempo real (Chainlink Data Feeds) para gestionar el riesgo financiero mediante límites denominados en Dólares (USD).

---

## Despliegue Oficial

El contrato se encuentra desplegado y **verificado** en la testnet de Sepolia.

| Contrato | Dirección | Explorador |
|----------|-----------|------------|
| **KipuBankV2** | `0x20D804430f96D8646dE6E95D777e5F6aD37D108F` | [Ver código en Etherscan](https://sepolia.etherscan.io/address/0x20D804430f96D8646dE6E95D777e5F6aD37D108F#code) |

---

## Arquitectura y Mejoras (V1 vs V2)

Esta versión abandona la rigidez de un contrato básico para adoptar patrones de composición y seguridad estándar en la industria DeFi.

| Característica | Implementación V1 (Ingenua) | Implementación V2 (Producción) | Motivo de la Mejora |
| :--- | :--- | :--- | :--- |
| **Soporte de Activos** | Moneda única (Solo ETH). | **Multi-Token (ETH + ERC20).** | Permite interoperabilidad real. Se utiliza `SafeERC20` para manejar tokens con implementaciones no estándar. |
| **Gestión de Riesgo** | Cap fijo en cantidad de tokens. | **Cap dinámico en USD ($).** | Un límite fijo (ej. 10 ETH) es ineficiente si el precio del activo se dispara. Usamos Oráculos para normalizar el riesgo a valor fiat. |
| **Gobernanza** | `Ownable` (Dueño único). | **AccessControl (RBAC).** | Seguridad granular. El rol `ADMIN` puede ajustar parámetros críticos sin tener custodia de los fondos. |
| **Seguridad** | Validaciones manuales. | **ReentrancyGuard + Custom Errors.** | Protección pasiva contra ataques de reentrada y optimización de gas en el despliegue. |

---

## Decisiones de Diseño y Trade-offs

Durante la refactorización, se tomaron decisiones conscientes priorizando la seguridad y la eficiencia del gas sobre la precisión absoluta del TVL.

### 1. Prevención de DoS en el Bank Cap
* **Problema:** Para calcular el TVL (Total Value Locked) exacto del banco en cada transacción, sería necesario iterar sobre todos los depósitos y consultar el precio de cada token. Esto costaría demasiado gas (O(n)), haciendo el contrato vulnerable a ataques de Denegación de Servicio.
* **Solución:** Implementamos una verificación "optimista". La función `_checkBankCap` valida si el **depósito entrante** (convertido a USD) excede el espacio disponible, sin recalcular todo el historial.

### 2. Normalización de Decimales
* Los oráculos de Chainlink devuelven 8 decimales, mientras que los tokens ERC-20 varían (USDC=6, DAI=18).
* **Diseño:** El contrato normaliza internamente todos los valores a **18 decimales** (`TARGET_DECIMALS`) antes de cualquier comparación lógica, garantizando consistencia matemática.

### 3. Protección contra Precios Desactualizados ("Stale Prices")
* No se confía ciegamente en el oráculo. Se implementó una validación de tiempo (`updatedAt`) que revierte la transacción si el precio de Chainlink tiene más de 1 hora de antigüedad, protegiendo al protocolo contra arbitrajes durante caídas de la red del oráculo.

---

## 🛠

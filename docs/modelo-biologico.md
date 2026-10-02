# Modelo biológico educativo

## Variables y unidades

DBO₅ en mg/L; NH₄-N, NO₂-N y NO₃-N en mg N/L; DO en mg O₂/L; XH, XAOB y XNOB en mg SSTV activa/L. Q en m³/d, V en m³, SRT y tiempo en días, KLa y μ/b en d⁻¹. YH en kg SSTV/kg DBO₅ y YAOB/YNOB en kg SSTV/kg N. La conversión es `masa[kg]=C[mg/L]·V[m³]/1000` y `carga[kg/d]=Q·C/1000`. Funciones centralizadas en `units.ts`.

## Cinéticas y balances

Para i = H, AOB, NOB:

$$\mu_i=\mu_{i,max}\theta_i^{T-T_{ref}}\frac{S_i}{K_i+S_i}\frac{DO}{K_{O,i}+DO}$$

S_H = DBO₅, S_AOB = NH₄-N, S_NOB = NO₂-N. μ está en d⁻¹. θ es adimensional. La corrección de temperatura se aplica al crecimiento, no al decaimiento en esta versión.

$$\frac{dX_i}{dt}=(\mu_i-b_i-1/SRT)X_i$$

Separación ideal retiene biomasa; el caudal no arrastra X directamente. Las pérdidas por purga se controlan mediante SRT, sin confundirla con HRT=V/Q. X=0 permanece cero. El washout es un balance negativo mantenido, no una animación programada.

Sea rH=μH XH/YH, rA=μAOB XAOB/YAOB, rN=μNOB XNOB/YNOB:

$$dDBO/dt=(Q/V)(DBO_{in}-DBO)-r_H$$
$$dNH_4/dt=(Q/V)(NH_{4,in}-NH_4)-r_A$$
$$dNO_2/dt=(Q/V)(NO_{2,in}-NO_2)+r_A-r_N$$
$$dNO_3/dt=(Q/V)(NO_{3,in}-NO_3)+r_N$$

La suma de las reacciones de N es exactamente cero. No se modela asimilación de N en biomasa, por lo que este balance representa N inorgánico disuelto, no todos los componentes de una EDAR real.

$$OUR=f_Cr_H+3.43r_A+1.14r_N$$
$$dDO/dt=KLa(DO_{sat}(T)-DO)-(Q/V)DO-OUR$$

Entrada DO=0. DOsat(T)=14,652−0,41022T+0,007991T²−0,000077774T³, aproximación de agua dulce a presión cercana al nivel del mar, usada en 5–35 °C. En DO impuesto se omite el cálculo dinámico y se registra el suministro necesario para cerrar el balance. Un ajuste instantáneo de DO equivale a suministro/retiro externo ideal.

Alcalinidad consumida = 7,14·N oxidado por AOB, como CaCO₃. No se infiere pH ni se limita reacción por alcalinidad. Oxidación completa AOB+NOB: 4,57 kg O₂/kg N, sin corrección por asimilación que el modelo omite.

## Integración positiva y conservativa

Cada subpaso usa intercambio hidráulico exacto `ΔC=(Cin−C)(1−exp(−QΔt/V))`. La salida media del paso se calcula como entrada menos intercambio neto para conservar masa, aunque difiera ligeramente de Q·C final. Aireación y pérdidas de biomasa usan exponentiales. El subpaso máximo es 0,005 días, reducido para SRT corta; se registran resultados cada 0,1 días y exactamente en los eventos.

La depleción de cada sustrato usa Monod implícito. Con capacidad c=μmax·factorT·factorDO·X·Δt/Y, se resuelve `S' + c S'/(K+S') = S`. La raíz positiva es `S'=2KS/(sqrt((K+c−S)²+4KS)+K+c−S)`. Esto limita consumo al inventario y evita concentraciones negativas o equilibrio espurio en cero causado por recortar Euler. NO₂ producido por AOB queda disponible para NOB en el siguiente subpaso. Si la suma de consumo O₂ supera el inventario, todas las reacciones se reducen proporcionalmente. μ mostrada es el crecimiento realizado por unidad de biomasa, coherente con ese límite.

No se usa un clamp de concentraciones para ocultar pérdidas de masa. Se calcula residuo de N, DBO y O₂ en cada subpaso y se muestran comprobaciones de 1e−7 kg en modo desarrollador. El historial conserva el residuo del último subpaso de cada intervalo; los tests de reactor cerrado también verifican conservación global. Los estados y exportaciones usan los mismos resultados.

Cambiar volumen conserva masa de solutos y biomasa ajustando concentración con V anterior/V nuevo. Es una perturbación ideal geométrica; no representa llenado con agua de composición específica. Los cambios operativos conservan la trayectoria previa al evento y actualizan condiciones a partir del mismo día. El balance del paso del evento puede incluir suministro externo para DO fijo.

## Interpretación

Estados poblacionales usan crecimiento realizado, decaimiento, 1/SRT e historial de cinco días. Riesgo de washout: balance negativo que proyecta una reducción superior al 75 % en diez días. Desaparición práctica: X<0,01 mg/L, umbral de representación, nunca se fuerza X=0. Recuperación: crecimiento positivo después de un periodo negativo. «Estable» admite margen 0,005 d⁻¹. Estos márgenes se declaran como indicadores educativos.

La eficiencia mostrada es eliminación aparente `(NH4in−NH4)/NH4in`; puede ser negativa durante descarga de inventario previo y no equivale necesariamente a la conversión instantánea por AOB. El informe declara criterios de pérdida/recuperación; no determina cumplimiento normativo.

## Límites

No hay desnitrificación, asimilación, inhibición, limitación de nutrientes, pH, mantenimiento, demanda endógena ni residuos inertes. b constante. La demanda carbonácea usa el factor del Excel y no cierra carbono elemental. La función de crecimiento es educativa; parámetros nuevos deben calibrarse. Esta implementación no es ASM1 ni ASM2d.

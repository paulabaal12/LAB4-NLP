# LAB4-NLP
---
## 1. ¿El modelo parece apropiado para reseñas en español?
Sí, parcialmente, y con una diferencia importante entre los dos modelos.

El modelo de sentimiento sí es apropiado en el sentido básico, por ejemplo: "La comida taba bien rika pero el serbicio pesimo, nunka volvemos". Fue clasificada NEGATIVE con 0.9247 pese a tener cuatro errores de escritura. Eso es mérito del preentrenamiento sobre texto informal de redes sociales, donde ese tipo de escritura abunda.

Sin embargo, hay un desajuste de dominio que sí importa. El modelo fue entrenado con tweets, no con reseñas de restaurantes. Los tweets son cortos, expresivos y suelen tener una sola polaridad; las reseñas son más largas y casi siempre evalúan varios aspectos a la vez (comida, servicio, precio, ambiente). Cuando esos aspectos apuntan en direcciones distintas, el modelo se ve obligado a comprimir todo en una sola etiqueta y pierde información. Eso se observa en las reseñas 3, 8, 21 y 25.

## 2. ¿Qué errores de sentimiento encontraste?
Se identificaron varios errores:  
  
1. **Sarcasmo:** La reseña 14: "Qué maravilla, dos horas de espera en El Portal del Ángel para que me sirvieran arroz frío. Simplemente espectacular." fue clasificada POSITIVE con score 0.9577. Un humano la lee como claramente negativa. El modelo se ancló en el léxico superficial ("maravilla", "espectacular") sin resolver la contradicción con el contenido factual de la queja 

2. **Opiniones mixtas colapsadas a una sola polaridad**: "Ambiente hermoso, pero precios muy altos" -> Predicción Positive y es mixta o "Café rico pero carísimo" -> NEGATIVE y de igual forma es mixta.

3. **Uso deficiente de la clase NEUTRAL**: La reseña 15: "Rico." es claramente positiva y salió NEUTRAL con 0.4480.

4. **Negación**: "No fue una mala experiencia... me sorprendió gratamente" fue correctamente clasificada POSITIVE con 0.9717. Pero la reseña 23 "No puedo decir que fue mala la cena en Los Girasoles, de hecho me encantó cada platillo" que tiene doble negación, acertó la etiqueta con un score de apenas 0.3820. Acertó, pero prácticamente por azar.

## 3. ¿Qué problemas observaste en NER?  

1. **Fragmentación de entidades por subtokens**: Pese a usar aggregation_strategy="simple", muchas entidades quedaron partidas en pedazos de wordpiece. En la reseña 1: Sab(LOC) + ##or Maya(LOC) en lugar de Sabor Maya y en la reseña 6: Ca(ORG) + ##fete(ORG) + ##ría Luna(ORG) en lugar de Cafetería Luna.

2. **Confusión sistemática ORG y LOC**: 

| Entidad | Etiqueta Asignada | Etiqueta Correcta |
| ------- | ------- | ------- |
|El Fogón Chapín| LOC | ORG |
|Comedor de Doña Rosa| LOC | ORG |
|Taquería El Compadre| ORG | ORG |

Esto explica por qué LOC domina con 31 ocurrencias frente a 17 de ORG. buena parte de esos LOC son en realidad organizaciones.

## 4. ¿Cómo influye el score en tu confianza?

El score es una señal útil pero asimétrica y no calibrada, y conviene tratarlo como tal. Donde sí funciona. Los scores bajos fueron buenos detectores de dificultad. Las cinco reseñas de menor confianza son precisamente casos genuinamente ambiguos: doble negación, texto de una palabra, opinión tibia y opinión mixta. Es decir, un score bajo es una alerta confiable de que hay que revisar manualmente. Donde no funciona. Un score alto no garantiza que la predicción sea correcta.  
La razón técnica es que el score es simplemente el softmax sobre los logits de la última capa, mide qué tan consistente es la entrada con los patrones aprendidos, no qué tan probable es que la respuesta sea verdadera.

## 5. ¿Qué datos etiquetarías manualmente para calcular precision, recall y F1?

Para sentimiento. Un conjunto de al menos 300–500 reseñas del mismo dominio, etiquetadas manualmente con el esquema POSITIVE / NEGATIVE / NEUTRAL, y balanceado. No replicando el sesgo 19/11/2 de este conjunto, sino con representación suficiente de cada clase. Para que el recall de NEUTRAL sea medible. Incluiría deliberadamente subconjuntos de casos difíciles (sarcasmo, negación, opiniones mixtas) marcados con una columna adicional de categoría, para poder reportar F1 desagregado por tipo de dificultad y no solo un promedio global que los oculte. Añadiría también una columna de sentimiento por aspecto (comida / servicio / precio / ambiente), porque es la única forma honesta de evaluar reseñas mixtas.  

## 6. ¿Usarías estas predicciones para tomar decisiones automáticas importantes? ¿Por qué?

No, no para decisiones importantes y automáticas. La reseña 14 es el argumento decisivo: un cliente describe dos horas de espera y comida fría, y el sistema lo registra como opinión positiva con 0.96 de confianza. Ningún umbral automático habría detenido ese error. Si esas predicciones alimentaran un tablero de satisfacción del cliente, la queja desaparecería del reporte. Ademas, las opiniones mixtas y sarcásticas se colapsan de manera predecible, y NER confunde ORG con LOC de forma consistente.
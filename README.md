# Pruebas
Guía de pensamiento para la para la realización de pruebas de software.


## Pruebas Unitarias

La escritura de pruebas unitarias reporta muchos beneficios; las pruebas ayudan con la regresión, proporcionan documentación y facilitan un buen diseño.

- **Menos tiempo para realizar pruebas funcionales**.
- **Protección frente a regresión**: Es habitual que los evaluadores no solo prueben una nueva característica, sino también las ya existentes con el fin de comprobar que siguen funcionando según lo previsto
- **Documentación ejecutable**:Si tiene un conjunto de pruebas unitarias con un nombre adecuado, cada prueba debe ser capaz de explicar con claridad el resultado esperado de una acción determinada. Además, debe ser capaz de comprobar que funciona.
- **Menos código acoplado**: Sin crear pruebas unitarias para el código que se está escribiendo, el acoplamiento puede ser menos evidente. Por lo que al escribir pruebas para el código, este se desacopla naturalmente, ya que, de otra forma, sería más difícil de probar.


### Características de una buena prueba unitaria

- **Rápida**. Debe ser rápida, su ejecución debería durar milisegundos.
- **Aislada**. No tienen ninguna dependencia con algún factor externo, como sistemas de archivos o bases de datos.
- **Reiterativa**. Debe devolver siempre el mismo resultado, si no cambia nada entre ejecuciones.
- **Autocomprobada**. La prueba debe ser capaz de detectar automáticamente si el resultado ha sido correcto o incorrecto sin necesidad de intervención humana.
- **Oportuna**. Una prueba unitaria no debe tardar un tiempo desproporcionado en escribirse en comparación con el código que se va a probar. Si observa que la prueba del código tarda mucho en comparación con su escritura, considere un diseño más fácil de probar.


### Conceptos básicos

- *System* (Sistema): Bloque de código envuelto en un como método publico que está sujeto a prueba. 
- _Fake_ (Emulación): Una emulación es un término genérico que se puede usar para describir un _stub_ o _mock_.
- _Mock_ (Objeto ficticio): un _mock_ es una emulación del sistema que decide si una prueba unitaria se ha superado o no.
    ```C#
    var mockOrder = new FakeOrder();
    var purchase = new Purchase(mockOrder);

    purchase.ValidateOrders();

    // Decide si una prueba se ha superado o no
    Assert.True(mockOrder.Validated); 
    ```
- _Stub_: un _stub_ es un reemplazo controlable para una dependencia existente (o colaborador) en el sistema. Con un _stub_, puede probar el código sin tratar directamente con la dependencia. De forma predeterminada, una emulación empieza como un _stub_.

    ```csharp
    // Reemplazo controlable para una dependencia
    var stubOrder = new FakeOrder();
    var purchase = new Purchase(stubOrder); 

    purchase.ValidateOrders();

    Assert.True(purchase.CanBeShipped);
    ```
- *Seam* (arreglo): Arreglo realizado a *System*  con el fin de controlar la salida de métodos estaticos desde la prueba unitaria mediante la creación de stub.

### Procedimientos recomendados

1. Asignar nombre a las pruebas: El nombre de la prueba debe presentarse como una declaración o hecho de la vida que exprese flujos de trabajo y resultados. Se recomienda:

- Utilizar formato "Snake Case" al nombrar la prueba.
- El nombre de la prueba deberá estar compuesto de tres partes: el nombre del método que se va a probar **(method)**, el comportamiento esperado al invocar al escenario **(expected)**  y el escenario en el que se está probando **(condition)**: 

    ```
    <method>_Should<expected>_When<condition>
    ```

    *Consideraciones*: dependiendo del contexto, es posible que desee sustituir los verbos *should/when* por algo más apropiado.

    *Microsoft* recomienda el siguiente formato:
    
    ```
    <method>_<condition>_<expected>
    ```

1. Organizar las pruebas: 

    **Arrange**, **Act**, **Assert** es un patrón común al realizar pruebas unitarias (AAA). Como el propio nombre implica, consta de tres acciones principales:

    - **Arrange** (organizar): organizar los objetos, crearlos y configurarlos según sea necesario.
    - **Act** (actuar): Actuar en un objeto.
    - **Assert**   (aserción): Declarar que algo es como se espera.

    Aplicando el patrón AAA, ganaremos legibilidad.

    ```C#
        [Fact]
        public void Add_EmptyString_ReturnsZero()
        {
            // Arrange
            var stringCalculator = new StringCalculator();

            // Act
            var actual = stringCalculator.Add("");

            // Assert
            Assert.Equal(0, actual);
        }
    ```

2. Escribir pruebas correctas lo más sencillas posible: 

    - Al escribir pruebas, debemos centrarnos en el comportamiento. 
    - El establecimiento de propiedades adicionales en los modelos o el empleo de valores distintos de cero cuando no es necesario solo resta de lo que se quiere probar.

    **Importante**: Las pruebas que incluyen más información de la necesaria para superarse tienen una mayor posibilidad de incorporar errores en la prueba y pueden hacer confusa su intención.

3. Evitar cadenas mágicas:

   - Los nombres de variable no pueden ser genéricos.
   - Los nombres de las variables deben expresar la entrada y el estado esperado. Ejemplo: BAD_DATA, EMPTY_ARRAY o NON_INITIALIZED_PERSON.
   - No necesita declararlas como CONST, pero el nombre debe representar la intención del autor de la prueba.

    **Importante** Una buena manera de verificar si sus nombres son lo suficientemente buenos es mirar el *Assert* al final del método de prueba. Si la línea *Assert* expresa cuál es su requerimiento, o se acerca, probablemente esté allí.

    ```C#
    Assert.AreEqual(-1, val) // No es semantico
    ```

    ```C#
    Assert.AreEqual(BAD_INITIALIZATION_CODE, ReturnCode) //Semantico
    ```

4. Evitar la lógica en las pruebas:
   
   Al escribir las pruebas unitarias, evite la concatenación de cadenas manual y las condiciones lógicas como if, while, for, switch, etc.

5. Tenga preferencias en usar Helper o metodos auxiliares para el montaje (setup) y desmontaje de la prueba.


    ```C#
    private readonly StringCalculator stringCalculator;
    public StringCalculatorTests()
    {
        stringCalculator = new StringCalculator();
    }

    [Fact]
    public void Add_TwoNumbers_ReturnsSumOfNumbers()
    {
        var result = stringCalculator.Add("0,1");

        Assert.Equal(1, result);
    }

    ```

    ```C#
    [Fact]
    public void Add_TwoNumbers_ReturnsSumOfNumbers()
    {
        var stringCalculator = CreateDefaultStringCalculator();

        var actual = stringCalculator.Add("0,1");

        Assert.Equal(1, actual);
    }

    private StringCalculator CreateDefaultStringCalculator()
    {
        return new StringCalculator();
    }
    ```

6. Evitar varias aserciones: 
   
   Al escribir las pruebas, intente incluir solo una aserción por prueba. Si tiene la intención de colocar más de una aserción en la prueba, antes considere:

   - Crear una prueba independiente para cada aserción.
   - Usar pruebas con parámetros.

7. Validar métodos privados mediante la prueba unitaria de métodos públicos:
   
   En la mayoría de los casos, no debería haber necesidad de probar un método privado. Los métodos privados son un detalle de implementación.

8. Referencias estáticas de stub:

    Para resolver este problema, deberá incorporar un arreglo en el código de producción. Primero deberá encapsular el código que necesita controlar en una interfaz y luego, hacer que el código de producción dependa de esta interfaz.
    
    Dado este caso:

    ```C#
    public int GetDiscountedPrice(int price)
    {
        if(DateTime.Now.DayOfWeek == DayOfWeek.Tuesday)
        {
            return price / 2;
        }
        else
        {
            return price;
        }
    }
    ```

    En el enfoque anterior no se puede tener el control del resultado de la ejecución del método estatico *Now* de la clase *DateTime* desde la prueba unitaria generando el stub correspondiente, por lo que tendrá que realizar un arreglo al *system*, como sigue:

    ```C#
    public interface IDateTimeProvider
    {
        DayOfWeek DayOfWeek();
    }

    public int GetDiscountedPrice(int price, IDateTimeProvider dateTimeProvider)
    {
        if(dateTimeProvider.DayOfWeek() == DayOfWeek.Tuesday)
        {
            return price / 2;
        }
        else
        {
            return price;
        }
    }
    ```

    Y desde la prueba, crear el stub correpondiente como veremos a continuación:

    ```C#
    public void GetDiscountedPrice_ByDefault_ReturnsFullPrice()
    {
        var priceCalculator = new PriceCalculator();
        var dateTimeProviderStub = new Mock<IDateTimeProvider>();
        dateTimeProviderStub.Setup(dtp => dtp.DayOfWeek()).Returns(DayOfWeek.Monday);

        var actual = priceCalculator.GetDiscountedPrice(2, dateTimeProviderStub);

        Assert.Equals(2, actual);
    }
    ```

    **Importante**: Uno de los principios de una prueba unitaria es que debe tener control total del sistema sometido a prueba. Esto puede ser problemático cuando el código de producción incluye llamadas a referencias estáticas com vimos en el ejemplo anterior.
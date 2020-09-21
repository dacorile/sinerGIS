# SinerGIS
Solución planteada por el grupo sonerGIS para el problema de Control de ocupación de espacio en grandes superficies ante el problema humanitario, de salud y logístico que presenta la pandemia del COVID-19.

<!DOCTYPE html>
<html>

<head>
    <script src="https://code.jquery.com/jquery-3.5.1.slim.min.js" integrity="sha384-DfXdz2htPH0lsSSs5nCTpuj/zy4C+OGpamoFVy38MVBnE+IbbVYUew+OrCXaRkfj" crossorigin="anonymous"></script>
    <script src="https://cdn.jsdelivr.net/npm/popper.js@1.16.1/dist/umd/popper.min.js" integrity="sha384-9/reFTGAW83EW2RDu2S0VKaIzap3H66lZH81PoYlFhbGU+6BZp6G7niu735Sk7lN" crossorigin="anonymous"></script>
    <script src="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/js/bootstrap.min.js" integrity="sha384-B4gt1jrGC7Jh4AgTPSdUtOBvfO8shuf57BaghqFfPlYxofvL8/KUEfYiJOMMV+rV" crossorigin="anonymous"></script>
    <script type="text/javascript" src="https://kit.fontawesome.com/bea5739c92.js" crossorigin="anonymous"></script>
    <script src="https://unpkg.com/driver.js/dist/driver.min.js"></script>
    <script type="text/javascript">
        function getValueFromSelect(id1, id2, id3) {
            var genero = document.getElementById(id1).value;
            console.log(genero);
            var edad = parseInt(document.getElementById(id2).value);
            console.log(edad);
            //var q_factores_riesgo = parseInt(document.getElementById(id3).value);
            var q_factores_riesgo = 0;
            console.log(q_factores_riesgo);
            var nivel_de_riesgo = "bajo";
            var cantidad_factores = [];
            for (var i = 1; i < 6; i++) {
                var x = document.getElementById("ch" + i.toString()).checked.toString();
                if (x == 'true') {
                    cantidad_factores.push(x);
                }
            }
            q_factores_riesgo = parseInt(cantidad_factores.length);

            if (edad > 34) {
                if (q_factores_riesgo <= 1) {
                    if (edad > 50) {
                        alert("Factor de riesgo: alto")
                    } else {
                        alert("Factor de riesgo: medio");
                    }
                } else if (q_factores_riesgo >= 2) {
                    alert("Factor de riesgo: alto")
                } else {
                    alert("Factor de riesgo: medio")
                }

            } else {
                if (q_factores_riesgo > 2) {
                    console.log("Factores: " + q_factores_riesgo)
                    console.log(cantidad_factores)
                    alert("Factor de riesgo: alto")
                } else {
                    alert("Factor de riesgo: medio 6")
                }
            }

        }
    </script>
    <meta charset="utf-8" />
    <meta name="viewport" content="initial-scale=1,maximum-scale=1,user-scalable=no" />
    <title>Sistema de Monitoreo por Aglomeración</title>
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/css/bootstrap.min.css" integrity="sha384-JcKb8q3iqJ61gNV9KGb8thSsNjpSL0n8PARn9HuZOnIxN0hoP+VmmDGMN5t9UJ0Z" crossorigin="anonymous" />

    <link rel="stylesheet" href="https://js.arcgis.com/4.16/esri/themes/light/main.css" />

    <style>
        html,
        body,
        #viewDiv {
            padding: 0;
            margin: 0;
            height: 97.6%;
            width: 100%;
        }
        
        .globalbtn {
            position: absolute !important;
            z-index: 1;
            padding: 11px 20px 10px 15px;
            border-radius: 0px 25px 25px 0px;
            color: white;
            border: none;
            background-color: #2e668b;
            font-size: 20px;
            margin-top: 170px;
        }
        
        .settingsbtn {
            position: absolute !important;
            z-index: 1;
            padding: 11px 15px 10px 20px;
            border-radius: 25px 0px 0px 25px;
            color: white;
            border: none;
            right: 0;
            background-color: #2e668b;
            font-size: 20px;
            margin-top: 100px;
        }
        
        #layerToggle {
            top: 20px;
            right: 20px;
            position: absolute;
            z-index: 99;
            background-color: white;
            border-radius: 8px;
            padding: 10px;
            opacity: 0.75;
        }
        
        #slide {
            position: fixed;
            background: #fff;
            width: 230px;
            left: -230px;
            height: auto;
            transition: left 0.4s ease-in-out;
            -o-transition: left 0.4s ease-in-out;
            -ms-transition: left 0.4s ease-in-out;
            -moz-transition: left 0.4s ease-in-out;
            -webkit-transition: left 0.4s ease-in-out;
            z-index: 99;
            margin-top: 100px;
            border-bottom-right-radius: 25px;
        }
        
        #toggle {
            position: absolute;
            left: 230px;
            background-color: #2e668b;
            z-index: 99;
            font-size: 20px;
            padding: 11px 20px 10px 15px;
            border-radius: 0px 25px 25px 0px;
            color: white;
        }
        
        .box {
            padding: 20px;
            z-index: 99;
        }
        
        #slide:hover {
            left: 0px;
            z-index: 99;
        }
        
        #optionsDiv {
            background-color: white;
            color: black;
            padding: 6px;
            max-width: 400px;
        }
    </style>

    <script src="https://js.arcgis.com/4.16/"></script>
    <script>
        require([
            "esri/Map",
            "esri/views/MapView",
            "esri/Graphic",
            "esri/layers/FeatureLayer",
            "esri/widgets/FeatureTable",
            "esri/widgets/BasemapToggle",
            "esri/widgets/Legend",
            "esri/widgets/Search",
            "esri/widgets/Expand",
            "esri/smartMapping/renderers/heatmap",
            "esri/core/watchUtils"

        ], function(
            Map,
            MapView,
            Legend,
            FeatureLayer,
            FeatureTable,
            BasemapToggle,
            Graphic,
            Search,
            Expand,
            heatmapRendererCreator,
            watchUtils

        ) {
            var map = new Map({
                basemap: "osm",
            });

            var view = new MapView({
                container: "viewDiv",
                map: map,
                extent: {
                    // autocasts as new Extent()
                    xmin: -9988935.74372658,
                    ymin: 274005.980628594,
                    xmax: -6742431.49129105,
                    ymax: 1051247.88539887,
                    spatialReference: 102100,
                },
            });

            var templateAeropuerto = {
                title: "<i class='fas fa-plane-departure mr-2'></i> {Aeropuerto}",
                content: `
                          <ul class='list-group  list-group-flush'>
                            <li class='list-group-item'>Id: {SKAR}</li>
                            <li class='list-group-item'>Tipo: {Internacio}</li>
                            <li class='list-group-item'>Departamento: { Quindío }</li>
                          </ul>
                          `,
            };

            var templateBiblioteca = {
                title: "<i class='fas fa-book-open mr-2'></i> {NOMBRE_DE_LA_BIBLIOTECA}",
                content: `
                          <ul class='list-group  list-group-flush'>
                            <li class='list-group-item'>Id: {OID}</li>
                            <li class='list-group-item'>Tipo: {TIPO_DE_BIBLIOTECA}</li>
                            <li class='list-group-item'>Departamento: { DEPARTAMENTO }</li>
                          </ul>
                          `,
            };

            var templateComerciales = {
                title: "<i class='fas fa-shopping-bag mr-2'></i> {Nombre_Cad}",
                content: `
                          <ul class='list-group  list-group-flush'>
                            <li class='list-group-item'>Id: {FID}</li>
                            <li class='list-group-item'>Tipo: {Categoria }</li>
                            <li class='list-group-item'>Departamento: { Departamen }</li>
                          </ul>
                          `,
            };

            var templateHotel = {
                title: "<i class='fas fa-bed mr-2'></i> {Nombre}",
                content: `
                          <ul class='list-group  list-group-flush'>
                            <li class='list-group-item'>Id: {FID}</li>
                            <li class='list-group-item'>Tipo: {Tipo}</li>
                            <li class='list-group-item'>Dirección: { Dirección }</li>
                          </ul>
                          `,
            };

            var templateMercado = {
                title: "<i class='fas fa-shopping-cart mr-2'></i> {Nombre_Cad}",
                content: `
                          <ul class='list-group  list-group-flush'>
                            <li class='list-group-item'>Id: {FID}</li>
                            <li class='list-group-item'>Tipo: {Categoria}</li>
                            <li class='list-group-item'>Departamento: { Departamen }</li>
                          </ul>
                          `,
            };

            var templateUniversidad = {
                title: "<i class='fas fa-graduation-cap mr-2'></i> {Nombre}",
                content: `
                          <ul class='list-group  list-group-flush'>
                            <li class='list-group-item'>Id: {FID}</li>
                            <li class='list-group-item'>Tipo: {Tipo}</li>
                            <li class='list-group-item'>Departamento: { Departamen }</li>
                          </ul>
                          `,
            };

            var templateReporte = {
                title: "Reporte",
                content: `
                          <ul class='list-group  list-group-flush'>
                            <li class='list-group-item'>Hora de Entrada: {ent }</li>
                            <li class='list-group-item'>Categoria - Codigo: {cat }</li>
                            <li class='list-group-item'>Departamento: { dep  }</li>
                            <li class='list-group-item'>Departamento: { mun }</li>
                          </ul>
                          `,
            };

            const renderer = {
                type: "heatmap",
                colorStops: [{
                    color: "rgba(63, 40, 102, 0)",
                    ratio: 0
                }, {
                    color: "#62001d",
                    ratio: 0.083
                }, {
                    color: "#62001d",
                    ratio: 0.166
                }, {
                    color: "#c60035",
                    ratio: 0.249
                }, {
                    color: "#c60035",
                    ratio: 0.332
                }, {
                    color: "#d80013",
                    ratio: 0.415
                }, {
                    color: "#d80013",
                    ratio: 0.498
                }, {
                    color: "#e3587b",
                    ratio: 0.581
                }, {
                    color: "#e3587b",
                    ratio: 0.664
                }, {
                    color: "#a46fbf",
                    ratio: 0.747
                }, {
                    color: "#c29f80",
                    ratio: 0.83
                }, {
                    color: "#e0cf40",
                    ratio: 0.913
                }, {
                    color: "#ffff00",
                    ratio: 1
                }],
                maxPixelIntensity: 25,
                minPixelIntensity: 0
            };

            // Aeropuertos
            var aeropuertos = new FeatureLayer({
                url: "https://services.arcgis.com/deQSb0Gn7gDPf3uV/arcgis/rest/services/MapaSinerGIS/FeatureServer/1",
                outFields: ["*"],
                popupTemplate: templateAeropuerto,
                visible: '',
            });
            // Bibliotecas
            var bibliotecas = new FeatureLayer({
                url: "https://services.arcgis.com/deQSb0Gn7gDPf3uV/arcgis/rest/services/MapaSinerGIS/FeatureServer/2",
                outFields: ["*"],
                popupTemplate: templateBiblioteca,
                visible: false,
            });
            // Centros Comerciales
            var centrosComerciales = new FeatureLayer({
                url: "https://services.arcgis.com/deQSb0Gn7gDPf3uV/arcgis/rest/services/MapaSinerGIS/FeatureServer/4",
                outFields: ["*"],
                popupTemplate: templateComerciales,
                visible: false,
            });
            // Hoteles Hostales
            var hoteles = new FeatureLayer({
                url: "https://services.arcgis.com/deQSb0Gn7gDPf3uV/arcgis/rest/services/MapaSinerGIS/FeatureServer/3",
                outFields: ["*"],
                popupTemplate: templateHotel,
                visible: false,
            });
            // Supermercados
            var mercados = new FeatureLayer({
                url: "https://services.arcgis.com/deQSb0Gn7gDPf3uV/arcgis/rest/services/MapaSinerGIS/FeatureServer/0",
                outFields: ["*"],
                popupTemplate: templateMercado,
                visible: false,
            });
            // Universidades
            var universidades = new FeatureLayer({
                url: "https://services.arcgis.com/deQSb0Gn7gDPf3uV/arcgis/rest/services/MapaSinerGIS/FeatureServer/5",
                outFields: ["*"],
                popupTemplate: templateUniversidad,
                visible: false,
            });

            var reporte = new FeatureLayer({
                url: "https://services.arcgis.com/deQSb0Gn7gDPf3uV/arcgis/rest/services/service_ebabb20f3fe74c1a802f74db61d01cb2/FeatureServer/0",
                outFields: ["*"],
                popupTemplate: templateReporte,
                renderer: renderer
            });

            var departments = new FeatureLayer({
                url: "https://services.arcgis.com/deQSb0Gn7gDPf3uV/arcgis/rest/services/Mapa_departamentos_de_Colombia/FeatureServer",
            });

            map.add(departments);
            map.add(aeropuertos);
            map.add(bibliotecas);
            map.add(centrosComerciales);
            map.add(hoteles);
            map.add(mercados);
            map.add(universidades);
            map.add(reporte);

            var aeropuertosLayerToggle = document.getElementById(
                "aeropuertosLayer"
            );
            var bibliotecasLayerToggle = document.getElementById(
                "bibliotecasLayer"
            );
            var centrosComercialesLayerToggle = document.getElementById(
                "centrosComercialesLayer"
            );
            var hotelesLayerToggle = document.getElementById("hotelesLayer");
            var mercadosLayerToggle = document.getElementById("mercadosLayer");
            var universidadesLayerToggle = document.getElementById(
                "universidadesLayer"
            );
            var reporteLayerToggle = document.getElementById("reporteLayer");

            aeropuertosLayerToggle.addEventListener("change", function() {
                aeropuertos.visible = aeropuertosLayerToggle.checked;
            });

            bibliotecasLayerToggle.addEventListener("change", function() {
                bibliotecas.visible = bibliotecasLayerToggle.checked;
            });

            centrosComercialesLayerToggle.addEventListener("change", function() {
                centrosComerciales.visible = centrosComercialesLayerToggle.checked;
            });

            hotelesLayerToggle.addEventListener("change", function() {
                hoteles.visible = hotelesLayerToggle.checked;
            });

            mercadosLayerToggle.addEventListener("change", function() {
                mercados.visible = mercadosLayerToggle.checked;
            });

            universidadesLayerToggle.addEventListener("change", function() {
                universidades.visible = universidadesLayerToggle.checked;
            });

            reporteLayerToggle.addEventListener("change", function() {
                reporte.visible = reporteLayerToggle.checked;
            });

            const sources = [{
                layer: aeropuertos,
                searchFields: ["Aeropuerto", "Quindío"],
                suggestionTemplate: "{Aeropuerto}",
                exactMatch: false,
                outFields: ["*"],
                name: "Aeropuertos",
                placeholder: "Aeropuerto el Dorado",
                zoomScale: 10000,
            }, {
                layer: bibliotecas,
                searchFields: [
                    "NOMBRE_DE_LA_BIBLIOTECA",
                    "DIRECCIÓN_DE_LA_BIBLIOTECA",
                ],
                suggestionTemplate: "{NOMBRE_DE_LA_BIBLIOTECA}",
                exactMatch: false,
                outFields: ["*"],
                name: "Bibliotecas",
                placeholder: "Biblioteca la Giralda",
                zoomScale: 10000,
            }, {
                layer: centrosComerciales,
                searchFields: ["Nombre_Cad", "Direccion"],
                suggestionTemplate: "{Nombre_Cad}",
                exactMatch: false,
                outFields: ["*"],
                name: "Centros Comerciales",
                placeholder: "Gran Estación",
                zoomScale: 10000,
            }, {
                layer: mercados,
                searchFields: ["Nombre_Cad", "Direccion"],
                suggestionTemplate: "{Nombre_Cad} - { Direccion }",
                exactMatch: false,
                outFields: ["*"],
                name: "Super Mercados",
                placeholder: "Justo y Bueno",
                zoomScale: 10000,
            }, {
                layer: universidades,
                searchFields: ["Nombre", "Dirección"],
                suggestionTemplate: "{Nombre}",
                exactMatch: false,
                outFields: ["*"],
                name: "Universidades",
                placeholder: "Universidad Distrital",
                zoomScale: 10000,
            }, {
                layer: hoteles,
                searchFields: ["Nombre", "Dirección"],
                suggestionTemplate: "{Nombre} - {Dirección}",
                exactMatch: false,
                outFields: ["*"],
                name: "Hoteles",
                placeholder: "Hotel Grand Hyatt",
                zoomScale: 10000,
            }, ];

            var toggle = new BasemapToggle({
                view: view,
                nextBasemap: "hybrid",
            });

            const searchWidget = new Search({
                view: view,
                allPlaceholder: "Buscar",
                sources: sources,
                includeDefaultSources: false,
            });



            const featureTableSuper = new FeatureTable({
                view: view,
                layer: mercados,
                fieldConfigs: [{
                    name: "Nombre_Cad",
                    label: "Nombre",
                }, {
                    name: "Departamen",
                    label: "Departamento",
                }, {
                    name: "Ciudad",
                    label: "Ciudad",
                }, {
                    name: "Direccion",
                    label: "Dirección",
                }, ],
                container: document.getElementById("tableDivSuper"),
            });

            const featureTableAeropuertos = new FeatureTable({
                view: view,
                layer: aeropuertos,
                fieldConfigs: [{
                    name: "Aeropuerto",
                    label: "Nombre",
                }, {
                    name: "Armenia_La",
                    label: "Departamento",
                }, {
                    name: "Quindío",
                    label: "Ciudad",
                }, {
                    name: "Internacio",
                    label: "Tipo",
                }, ],
                container: document.getElementById("tableDivAeropuertos"),
            });

            const featureTableBibliotecas = new FeatureTable({
                view: view,
                layer: bibliotecas,
                fieldConfigs: [{
                    name: "NOMBRE_DE_LA_BIBLIOTECA",
                    label: "Nombre",
                }, {
                    name: "DEPARTAMENTO",
                    label: "Departamento",
                }, {
                    name: "MUNICIPIO",
                    label: "Ciudad",
                }, {
                    name: "DIRECCIÓN_DE_LA_BIBLIOTECA",
                    label: "Dirección",
                }, ],
                container: document.getElementById("tableDivBibliotecas"),
            });

            const featureTableHoteles = new FeatureTable({
                view: view,
                layer: hoteles,
                fieldConfigs: [{
                    name: "Nombre",
                    label: "Nombre",
                }, {
                    name: "Tipo",
                    label: "Tipo",
                }, {
                    name: "Ciudad",
                    label: "Ciudad",
                }, ],
                container: document.getElementById("tableDivHoteles"),
            });

            const featureTableUniversidades = new FeatureTable({
                view: view,
                layer: universidades,
                fieldConfigs: [{
                    name: "Nombre",
                    label: "Nombre",
                }, {
                    name: "Departamen",
                    label: "Departamento",
                }, {
                    name: "Ciudad",
                    label: "Ciudad",
                }, {
                    name: "Tipo",
                    label: "Tipo",
                }, ],
                container: document.getElementById("tableDivUni"),
            });

            const featureTableCentrosComerciales = new FeatureTable({
                view: view,
                layer: centrosComerciales,
                fieldConfigs: [{
                    name: "Nombre_Cad",
                    label: "Nombre",
                }, {
                    name: "Departamen",
                    label: "Departamento",
                }, {
                    name: "Ciudad",
                    label: "Ciudad",
                }, {
                    name: "Direccion",
                    label: "Dirección",
                }, ],
                container: document.getElementById("tableDivCC"),
            });

            const featureTableAglo = new FeatureTable({
                view: view,
                layer: reporte,
                fieldConfigs: [{
                        name: "ent",
                        label: "Fecha entrada",
                    }, {
                        name: "salida",
                        label: "Fecha salida",
                    },

                    {
                        name: "dep",
                        label: "Departamento",
                    }, {
                        name: "mun",
                        label: "Ciudad",
                    }, {
                        name: "cat",
                        label: "Categoria",
                    }, {
                        name: "type",
                        label: "Establecimiento",
                    },
                ],
                container: document.getElementById("tableDivAglo"),
            });

            view.ui.add(searchWidget, {
                position: "top-right",
                index: 2,
            });

            view.ui.add(toggle, "bottom-left");
            view.ui.add("optionsDiv", "bottom-right");
        });

        $(document).ready(function() {});
    </script>
</head>

<body class="calcite">
    <header class="navbar navbar-expand navbar-dark bg-dark flex-column flex-md-row bd-navbar">
        <h5 class="mb-0 mt-0 text-white">
            Sistema para el monitoreo de personas en zonas de aglomeración
        </h5>
        <ul class="navbar-nav ml-md-auto mt-2 mt-md-0 mt-lg-0 mt-xl-0 mr-5" style="font-size: 13px !important">
            <li class="nav-item pt-1 pb-1">
                <a class="nav-link p-0 pr-md-2 pr-lg-2 pr-xl-2 pl-md-2 pl-lg-2 pl-xl-2 text-white" href="./index.html"><i class="fas fa-globe-americas mr-2"></i>
            Visor
          </a>
            </li>
            <li class="nav-item pt-1 pb-1">
                <a class="nav-link p-0 pr-md-2 pr-lg-2 pr-xl-2 pl-md-2 pl-lg-2 pl-xl-2 text-white" href="./index2.html"><i class="fas fa-building mr-2"></i>Dashboard</a
          >
        </li>
        <li class="nav-item pt-1 pb-1">
          <a class="nav-link p-0 pr-md-2 pr-lg-2 pr-xl-2 pl-md-2 pl-lg-2 pl-xl-2 text-white" href="#staticBackdrop" data-toggle="modal" data-target="#staticBackdrop"><i class="far fa-address-book mr-2"></i>
         Encuesta
      </a>
            </li>
        </ul>
    </header>
    <button type="button" data-toggle="modal" data-target="#settingsModal" class="settingsbtn" id="settingsbtn">
      <i class="fa fa-cogs" aria-hidden="true"></i>
    </button>
    <button type="button" data-toggle="modal" data-target="#globalModal" class="globalbtn" id="globalbtn">
  <i class="fas fa-vial"></i>
  </button>


    <div id="slide">
        <div id="toggle"><i class="fas fa-layer-group"></i></div>
        <div class="box">
            <h5>Capas</h5>
            <div class="form-check">
                <input class="form-check-input" type="checkbox" id="aeropuertosLayer" />
                <label class="form-check-label" for="defaultCheck1">
            Aeropuertos
          </label>
            </div>
            <div class="form-check">
                <input class="form-check-input" type="checkbox" id="centrosComercialesLayer" />
                <label class="form-check-label" for="defaultCheck1">
            Centros Comerciales
          </label>
            </div>
            <div class="form-check">
                <input class="form-check-input" type="checkbox" id="bibliotecasLayer" />
                <label class="form-check-label" for="defaultCheck1">
            Bibliotecas
          </label>
            </div>
            <div class="form-check">
                <input class="form-check-input" type="checkbox" id="hotelesLayer" />
                <label class="form-check-label" for="defaultCheck1"> Hoteles </label>
            </div>
            <div class="form-check">
                <input class="form-check-input" type="checkbox" id="mercadosLayer" />
                <label class="form-check-label" for="defaultCheck1">
            Super Mercados
          </label>
            </div>
            <div class="form-check">
                <input class="form-check-input" type="checkbox" id="universidadesLayer" />
                <label class="form-check-label" for="defaultCheck1">
            Universidades
          </label>
            </div>
            <h5 class="mt-2">Analisis</h5>
            <div class="form-check">
                <input class="form-check-input" type="checkbox" id="reporteLayer" checked />
                <label class="form-check-label" for="defaultCheck1">
            Aglomeración
          </label>
            </div>
        </div>
    </div>

    <div id="optionsDiv" class="esri-widget" style="overflow: auto;height: 20rem">
        <h4 class="px-3"><strong>Leyenda</strong></h4>
        <img src="./legend.png"></img>
    </div>

    <div class="modal fade" id="staticBackdrop" data-backdrop="static" data-keyboard="false" tabindex="-1" aria-labelledby="staticBackdropLabel" aria-hidden="true">
        <div class="modal-dialog modal-dialog-centered">
            <div class="modal-content">
                <div class="modal-header">
                    <h5 class="modal-title" id="staticBackdropLabel">Escanea el codigo QR desde la app Survey123</h5>
                    <button type="button" class="close" data-dismiss="modal" aria-label="Close">
              <span aria-hidden="true">&times;</span>
            </button>
                </div>
                <div class="modal-body">
                    <div class="row">
                        <div class="col">
                            <img src="./qr.png" class="rounded mx-auto d-block" alt="..." style="max-width: 100%;">
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>


    <div class="modal fade" id="globalModal" tabindex="-1" aria-labelledby="globalModalLabel" aria-hidden="true">
        <div class="modal-dialog modal-dialog-centered">
            <div class="modal-content">
                <div class="modal-header">
                    <h5 class="modal-title" id="globalModalLabel">Riesgo de Contagio</h5>
                    <button type="button" class="close" data-dismiss="modal" aria-label="Close">
              <span aria-hidden="true">&times;</span>
            </button>
                </div>
                <div class="modal-body">
                    <div class="form-row">
                        <div class="col">
                            <select name="genero" id="genero" class="form-control">
                  <option selected disabled>Seleccione su genero</option>
                  <option value="M">Masculino</option>
                  <option value="F">Femenino</option>	      
               </select>
                        </div>
                        <div class="col">
                            <select name="edad" id="edad" class="form-control">
                  <option selected disabled>Seleccione su edad</option>
                <script type="text/javascript">
                for(var i=20; i<100;i++){
                  document.write("<option value=" + i +">"+ i +"</option>")
                }
                </script>	      
               </select>
                        </div>
                    </div>
                    <h5 class="mt-3">Factores de riesgo</h5>
                    <div class="row">
                        <div class="col">
                            <div class="form-check">
                                <input class="form-check-input" type="checkbox" value="" id="ch1">
                                <label class="form-check-label" for="defaultCheck1">
                    Hipertensión
                  </label>
                            </div>
                            <div class="form-check">
                                <input class="form-check-input" type="checkbox" id="ch2">
                                <label class="form-check-label" for="defaultCheck1">
                    Diabetes
                  </label>
                            </div>
                            <div class="form-check">
                                <input class="form-check-input" type="checkbox" id="ch3">
                                <label class="form-check-label" for="defaultCheck1">
                    Enfermedad Pulmonar Obstructiva Crónica (EPOC)
                  </label>
                            </div>
                            <div class="form-check">
                                <input class="form-check-input" type="checkbox" id="ch4">
                                <label class="form-check-label" for="defaultCheck1">
                    Enfermedad Renal Crónica
                  </label>
                            </div>
                            <div class="form-check">
                                <input class="form-check-input" type="checkbox" id="ch5">
                                <label class="form-check-label" for="defaultCheck1">
                    Inmunosupresión (por ejemplo cáncer, lupus, etc.)
                  </label>
                            </div>
                        </div>
                    </div>
                </div>
                <div class="modal-footer">
                    <button type="button" class="btn btn-dark" onclick="getValueFromSelect('genero','edad','q_fact' )">Consultar</button>
                </div>
            </div>
        </div>
    </div>


    <div class="modal fade" id="settingsModal" tabindex="-1" aria-labelledby="settingsModalLabel" aria-hidden="true">
        <div class="modal-dialog modal-xl modal-dialog-centered">
            <div class="modal-content">
                <div class="modal-header">
                    <h5 class="modal-title" id="settingsModalLabel">Información</h5>
                    <button type="button" class="close" data-dismiss="modal" aria-label="Close">
              <span aria-hidden="true"><i class="fas fa-times"></i></span>
            </button>
                </div>
                <div class="modal-body">
                    <ul class="nav nav-tabs" id="myTab" role="tablist">
                        <li class="nav-item" role="presentation">
                            <a class="nav-link active" id="aglo-tab" data-toggle="tab" href="#aglo" role="tab" aria-controls="contact" aria-selected="true">Aglomeración</a
                >
              </li>
              <li class="nav-item" role="presentation">
                <a
                  class="nav-link"
                  id="super-tab"
                  data-toggle="tab"
                  href="#super"
                  role="tab"
                  aria-controls="home"
                  aria-selected="false"
                  >Super Mercados</a
                >
              </li>
              <li class="nav-item" role="presentation">
                <a
                  class="nav-link"
                  id="aeropuertos-tab"
                  data-toggle="tab"
                  href="#aeropuertos"
                  role="tab"
                  aria-controls="profile"
                  aria-selected="false"
                  >Aeropuertos</a
                >
              </li>
              <li class="nav-item" role="presentation">
                <a
                  class="nav-link"
                  id="bibliotecas-tab"
                  data-toggle="tab"
                  href="#bibliotecas"
                  role="tab"
                  aria-controls="contact"
                  aria-selected="false"
                  >Bibliotecas</a
                >
              </li>
              <li class="nav-item" role="presentation">
                <a
                  class="nav-link"
                  id="hoteles-tab"
                  data-toggle="tab"
                  href="#hoteles"
                  role="tab"
                  aria-controls="contact"
                  aria-selected="false"
                  >Hoteles</a
                >
              </li>
              <li class="nav-item" role="presentation">
                <a
                  class="nav-link"
                  id="uni-tab"
                  data-toggle="tab"
                  href="#uni"
                  role="tab"
                  aria-controls="contact"
                  aria-selected="false"
                  >Universidades</a
                >
              </li>
              <li class="nav-item" role="presentation">
                <a
                  class="nav-link"
                  id="cc-tab"
                  data-toggle="tab"
                  href="#cc"
                  role="tab"
                  aria-controls="contact"
                  aria-selected="false"
                  >Centros Comerciales</a
                >
              </li>
            </ul>
            <div class="tab-content" id="myTabContent">
              <div
                class="tab-pane fade show active"
                id="aglo"
                role="tabpanel"
                aria-labelledby="aglo-tab"
              >
                <div id="tableDivAglo" style="height: 400px"></div>
              </div>
              <div
                class="tab-pane fade"
                id="super"
                role="tabpanel"
                aria-labelledby="super-tab"
              >
              <div id="tableDivSuper" style="height: 400px"></div>
              </div>
              <div
                class="tab-pane fade"
                id="aeropuertos"
                role="tabpanel"
                aria-labelledby="aeropuertos-tab"
              >
                
              <div id="tableDivAeropuertos" style="height: 400px"></div>
              </div>
              <div
                class="tab-pane fade"
                id="bibliotecas"
                role="tabpanel"
                aria-labelledby="bibliotecas-tab"
              >
                
              <div id="tableDivBibliotecas" style="height: 400px"></div>
              </div>
              <div
                class="tab-pane fade"
                id="hoteles"
                role="tabpanel"
                aria-labelledby="hoteles-tab"
              >
                
              <div id="tableDivHoteles" style="height: 400px"></div>
              </div>

              <div
                class="tab-pane fade"
                id="uni"
                role="tabpanel"
                aria-labelledby="uni-tab"
              >
                
              <div id="tableDivUni" style="height: 400px"></div>
              </div>

              <div
                class="tab-pane fade"
                id="cc"
                role="tabpanel"
                aria-labelledby="cc-tab"
              >
                
              <div id="tableDivCC" style="height: 400px"></div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
    <div id="viewDiv"></div>
  </body>
</html>

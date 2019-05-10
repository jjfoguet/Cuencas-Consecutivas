import arcpy
import os
arcpy.env.workspace = 'C:/Datos/Glaciares_vegas'
arcpy.env.overwriteOutput = True
carpeta = arcpy.env.workspace
outputFolder = 'C:/Datos/Glaciares_vegas/intermezzo'
### INPUTS

rios = 'C:/Datos/Glaciares_vegas/rios_unsplit.shp'
vegas = 'C:/Datos/Glaciares_vegas/vegas_orden.shp'
acum = 'C:/Datos/Glaciares_vegas/acum_lasconchas.tif'
flowdir = 'C:/Datos/Glaciares_vegas/acum_lasconchas.tif'

### RED DE RIOS SEGUN ORDEN Y EN UNA MISMA LINEA DE FLUJO

for orden in range(1,9,1):
    where = "GRID_CODE = {0}".format(orden)
    name = 'rios_{0}.shp'.format(orden)
    outputName = os.path.join(outputFolder, name)    
    subset = arcpy.Select_analysis(rios,outputName,where)
walk = arcpy.da.Walk(outputFolder,datatype="FeatureClass")
listaRios = []
for dir_path, dir_names, file_names in walk:
    for filename in file_names:
        listaRios.append(os.path.join(dir_path, filename))
listaFeatures = []
for i in range(0,len(listaRios),1):
                         inputs = listaRios[i:i+2]
                         j = i+1
                         nombreUnion = 'rios_{0}{1}'.format(j,j+1)
                         nombreSalida_union = os.path.join(carpeta,nombreUnion)
                         if len(inputs) < 2:
                             break
                         uniones = arcpy.FeatureToLine_management(inputs,nombreSalida_union)
                         whereSelect = "GRID_CODE = {0}".format(j)
                         nombreSelect = 'rios_{0}{1}select'.format(j, j+1)
                         nombreSalida_select = os.path.join(carpeta,nombreSelect)
                         ordenes = arcpy.Select_analysis(uniones,nombreSalida_select,whereSelect)
                         listaFeatures.append(ordenes)
riosProcesados = arcpy.Merge_management(listaFeatures,'C:/Datos/Glaciares_vegas/rios_procesados.shp')
riosProcesados = arcpy.AddField_management(featureTotal,"tramo","LONG")
riosProcesados = arcpy.CalculateField_management("rios_procesados","tramo","!FID! + 1","PYTHON")
### AGRUPACION DE LAS VEGAS SEGUN LOS TRAMOS DE LOS RIOS

## FUNCION PARA DEFINIR MAPPINGS para spatial join. DOS atributos (GRID_CODE y tramo, PIDIENDO max)
def mapping(target,join,fields,rule):
    fieldmappings = arcpy.FieldMappings()
    fieldmappings.addTable(target)
    fieldmappings.addTable(join)
    ordenFieldIndex = fieldmappings.findFieldMapIndex("{0}".format(fields[0]))
    tramoFieldIndex = fieldmappings.findFieldMapIndex("{0}".format(fields[1]))
    fieldmapOrden = fieldmappings.getFieldMap(ordenFieldIndex)
    fieldmapTramo = fieldmappings.getFieldMap(tramoFieldIndex)
    fieldOrden = fieldmapOrden.outputField
    fieldTramo = fieldmapTramo.outputField
    fieldmapOrden.mergeRule = rule
    fieldmapTramo.mergeRule = rule
    fieldmappings.replaceFieldMap(ordenFieldIndex, fieldmapOrden)
    fieldmappings.replaceFieldMap(tramoFieldIndex, fieldmapTramo)
    return fieldmappings

campos = ['GRID_CODE','tramo']
regla = 'max'
mapp = mapping(vegas,riosProcesados,campos,regla)
vegasJoin = arcpy.SpatialJoin_analysis(vegas, riosProcesados, os.path.join(carpeta,'vegasProcesadas.shp'), "#", "#", mapp)
vegasDissolve = arcpy.Dissolve_management(vegasJoin,'C:/Datos/Glaciares_vegas/vegas_dissolve.shp',"tramo")                                       

###IDENTIFICAR LOS POURPOINTS PARA CADA VEGA DISUELTA
zonal = arcpy.sa.ZonalStatistics(vegasDissolve,"tramo",acum,"MAXIMUM")
equal = arcpy.sa.EqualTo(zonal,acum)
setnull = arcpy.sa.SetNull(equal,1,"Value = 0")
pourPoints = arcpy.RasterToPoint_conversion(setnull,'C:/Datos/Glaciares_vegas/pourPoints.shp')
#delete gridcodefield
###PASAR LOS ATRIBUTOS DE TRAMO Y ORDEN A POURPOINT                           

mapp2 = mapping('pourPoints','vegas_procesadas',campos,regla)
pourAtribute = arcpy.SpatialJoin_analysis(pourPoints,vegasJoin,'C:/Datos/Glaciares_vegas/pour_atribute.shp',"JOIN_ONE_TO_ONE","KEEP_ALL",mapp2)

### CALCULO DE CUENCAS

# Análisis Metagenomica shotgun

## 1. **Creación del Directorio de Trabajo**

Primero, crea un directorio en tu sistema de archivos donde almacenarás los resultados de la evaluación de calidad:

```bash
mkdir -p /ruta/al/directorio/1_QC/1_raw_data_infor/
```

Aquí, `/ruta/al/directorio/` debe ser reemplazada con la ruta donde deseas guardar tus archivos.

## 2. **Instalación de fastp**

Si aún no tienes **fastp** instalado, puedes instalarlo utilizando **conda** o directamente desde GitHub:

**Conda:**

```bash
conda install -c bioconda fastp
```

**GitHub:**

```bash
git clone https://github.com/OpenGene/fastp
cd fastp
make
```

## 3. **Ejecución de fastp para la Evaluación de la Calidad**

Ejecuta **fastp** para cada muestra para generar los reportes de calidad. A continuación, te muestro cómo puedes hacerlo en un bucle de shell para las 11 muestras mencionadas:

```bash
for sample in CIS CM CTS DCS DSM TM NS DM OS DIM SIM
do
    fastp -i /ruta/a/las/muestras/${sample}_R1.fastq.gz \
          -I /ruta/a/las/muestras/${sample}_R2.fastq.gz \
          -o /ruta/al/directorio/1_QC/${sample}_R1_clean.fastq.gz \
          -O /ruta/al/directorio/1_QC/${sample}_R2_clean.fastq.gz \
          -j /ruta/al/directorio/1_QC/1_raw_data_infor/${sample}_fastp_report.json \
          -h /ruta/al/directorio/1_QC/1_raw_data_infor/${sample}_fastp_report.html \
          -w 4
done
```

En este script:

- **-i** y **-I** son las rutas a los archivos FASTQ de entrada (pares de lecturas).
- **-o** y **-O** son las rutas donde se guardarán los archivos FASTQ limpios.
- **-j** es la ruta para el archivo JSON que contiene un informe detallado.
- **-h** es la ruta para el informe HTML.
- **-w** especifica el número de hilos que **fastp** usará.

## 4. **Revisión de los Informes de Calidad**

Después de ejecutar **fastp**, revisa los archivos JSON y HTML generados en la carpeta `1_QC/1_raw_data_infor/`. Estos informes te proporcionarán detalles sobre la calidad de las secuencias y cualquier corrección realizada.

## 5. **Generación de Gráficos y Tablas**

Puedes consolidar las estadísticas en un archivo Excel para un resumen general:

```bash
# Combinar los JSON en un archivo CSV para revisión (opcional)
jq -s 'reduce .[] as $item ({}; . * $item)' /ruta/al/directorio/1_QC/1_raw_data_infor/*.json > /ruta/al/directorio/1_QC/1_raw_data_infor/All_raw_data_infor.json

# Convertir JSON a CSV
cat /ruta/al/directorio/1_QC/1_raw_data_infor/All_raw_data_infor.json | jq -r '[.summary.before_filtering, .summary.after_filtering] | (["Metric","RawData","CleanData"] | @csv), (.[] | [.total_reads, .total_bases, .q20_bases, .q30_bases, .gc_content] | @csv)' > /ruta/al/directorio/1_QC/1_raw_data_infor/All_raw_data_infor.csv
```

Esto generará un archivo CSV con un resumen de las estadísticas antes y después de la filtración.

---
title: "Implementando un operador Airflow para subir a Google Cloud Storage"
date: 2021-06-04T21:41:20-05:00
draft: true
publishdate: 2021-06-04T21:41:20-05:00
lastmod: 2021-06-04T21:41:20-05:00
tags: ["python", "airflow", "Composer", "Google", "Cloud", "Storage"]
type: "post"
comments: false
---
Apache Airflow es una plataforma que permite programar y monitoreas la ejecución de diferentes flujos de trabajo y está disponible como un servicio administrado en Google Cloud, con el nombre de Composer. Los flujos de trabajo se definen programáticamente utilizando archivos python en la forma de una secuencia de actividades enlazadas de tal mamera que forman un grafo dirigido en los que no se permite la existencia de ciclos (DAG del ingles Directed Acyclic Graph). Cada nodo del grafo es una actividad definida dentro de un elemento que en la plataforma toma el nombre de Operador.

La versión disponible en Google Cloud Composer incluye una amplia variedad de operadores que permiten realizar diferentes tareas dentro de la plataforma de Google Cloud, entre estas existen operadores para interactuar con servicios como Cloud Storage, Cloud BigQuery, Cloud Function y otros tantos servicios.

Hace unos días tuve la necesidad de crear un archivo en un bucket de Google Cloud Storage (GCS) y descubrí que entre los operadores disponibles no existía alguno que permitiera realizarlo. Por experiencias anteriores sabía que existe una clase llamada [`GCSHook`](https://github.com/apache/airflow/blob/9c792eabb9ed48a7b468e9a76b3662f20ad846cb/airflow/providers/google/cloud/hooks/gcs.py) que implementa varias operaciones relacionadas con GCS que no siempre tienen un operador asociado y en efecto la operación que necesitaba si estaba implementada en esa clase. Así que lo único que tuve que hacer fue crear un operador personalizado para exponer esa operación, la implementación que quedó es la siguiente:

{{<highlight python>}}
from typing import Optional, Sequence, Union
from airflow.providers.google.cloud.hooks.gcs import GCSHook
from airflow.models import BaseOperator

DEFAULT_TIMEOUT = 60

class GCSUploadObjectOperator(BaseOperator):
    """
    Sube un archivo local o datos en formato string o bytes a Google Cloud Storage.
    :param bucket_name: El bucket al que se subirá.
    :type bucket_name: str
    :param object_name: El nombre que tendrá el archivo subido.
    :type object_name: str
    :param filename: La ruta local al archivo que se subirá.
    :type filename: str
    :param data: Los datos del archivo en formato string o bytes que se subirá.
    :type data: str
    :param mime_type: El mimetype del archivo cuando se suba.
    :type mime_type: str
    :param gzip: Comprimir archivo local o datos de archivo para subirlo
    :type gzip: bool
    :param encoding: bytes encoding para los datos del archivo si se proporcionan en formato string
    :type encoding: str
    :param chunk_size: Tamaño de bloque de datos.
    :type chunk_size: int
    :param timeout: Request timeout en segundos.
    :type timeout: int
    :param num_max_attempts: Número de intentos para tratar de subir el archivo.
    :type num_max_attempts: int
    """
    def __init__(
        self,
        *,
        bucket_name: str,
        object_name: str,
        filename: Optional[str] = None,
        data: Optional[Union[str, bytes]] = None,
        mime_type: Optional[str] = None,
        gzip: bool = False,
        encoding: str = 'utf-8',
        chunk_size: Optional[int] = None,
        timeout: Optional[int] = DEFAULT_TIMEOUT,
        num_max_attempts: int = 1,
        gcp_conn_id: str = 'google_cloud_default',
        delegate_to: Optional[str] = None,
        impersonation_chain: Optional[Union[str, Sequence[str]]] = None,
        **kwargs
    ) -> None:
        super().__init__(**kwargs)
        self.bucket_name = bucket_name
        self.object_name = object_name
        self.filename = filename
        self.data = data
        self.mime_type = mime_type
        self.gzip = gzip
        self.encoding = encoding
        self.chunk_size = chunk_size
        self.timeout = timeout
        self.num_max_attempts = num_max_attempts
        self.gcp_conn_id = gcp_conn_id
        self.delegate_to = delegate_to
        self.impersonation_chain = impersonation_chain

    def execute(self, context) -> None:
        self.log.info(
            'Uploading to Cloud Storage: %s.',
            self.object_name)

        hook = GCSHook(
            google_cloud_storage_conn_id=self.gcp_conn_id,
            delegate_to=self.delegate_to,
            impersonation_chain=self.impersonation_chain,
        )

        hook.upload(
            bucket_name=self.bucket_name,
            object_name=self.object_name,
            filename=self.filename,
            data=self.data,
            mime_type=self.mime_type,
            gzip=self.gzip,
            encoding=self.encoding,
            chunk_size=self.chunk_size,
            timeout=self.timeout,
            #num_max_attempts=self.num_max_attempts
        )
{{</highlight>}}

De esta implementación la parte importante es la instanciación de la clase `GCSHook`, que es la que contiene todo lo necesario para crear un archivo en Cloud Storage y la invocación de la operación con la llamada `hook.upload`, a la cual le enviamos todos los parámetros requeridos los cuales podemos observar en código fuente de la clase.

Algo importante que mencionar es que la versión del código fuente observado incluye un parámetro `num_max_attempts`, sin embargo en la versión de Airflow que usé para desplegar mi DAG, este parámetro no estaba disponible por lo que quedó comentado. En otras versiones algunos otros parámetros podrían no estar disponibles y la manera segura de usar este operador es simplemente comentando esos parámetros.


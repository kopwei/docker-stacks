FROM kopkop/jupyter-tensorflow-notebook

RUN  pip install --quiet --no-cache-dir \
    jupyter_contrib_nbextensions && \
    fix-permissions "${CONDA_DIR}" && \
    fix-permissions "/home/${NB_USER}"

USER root
RUN /opt/conda/bin/jupyter contrib nbextension install 

USER ${NB_USER}

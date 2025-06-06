# This Dockerfile creates an enviroment / dependencies needed to run the
# neutronics-workshop.

# This Dockerfile can be build locally or a prebuild image can be downloaded

# To download the prebuild image
# docker pull ghcr.io/fusion-energy/neutronics-workshop

# To build this Dockerfile into a docker image:
# docker build -t ghcr.io/fusion-energy/neutronics-workshop -f .devcontainer/Dockerfile .

# to launch the docker image run with this command
# docker run -p 8888:8888 ghcr.io/fusion-energy/neutronics-workshop

# to push to github container repository, generate an access token
# https://github.com/settings/tokens

# login to ghcr
# echo ghp_yourgithubtoken | docker login ghcr.io -u yourgithubusername --password-stdin

# push the docker image to ghcr 
# docker push ghcr.io/fusion-energy/neutronics-workshop

FROM mcr.microsoft.com/devcontainers/base:bookworm AS dependencies

RUN apt-get --allow-releaseinfo-change update
RUN apt-get --yes update && apt-get --yes upgrade

# perhaps libnetcdf13 is needed for unstructured meshes in openmc
# RUN apt-get --yes install libnetcdf13

                          # eigen3 needed for DAGMC
RUN apt-get --yes install libeigen3-dev \
                          git \
                          wget \
                          mpich \
                          libmpich-dev \
                          libhdf5-serial-dev \
                          libhdf5-mpich-dev \
                          hdf5-tools \
                          # libnetcdf-dev is needed to allow NETCDF on MOAB which helps with tet meshes in OpenMC
                          libnetcdf-dev \
                          # libtbb-dev required for DAGMC
                          libtbb-dev \
                          # libglfw3-dev required for DAGMC
                          libglfw3-dev \
                          # needed for CadQuery functionality
                          libgl1-mesa-glx \
                          # needed for CadQuery functionality
                          libgl1-mesa-dev \
                          # needed for CadQuery functionality
                          libglu1-mesa-dev \
                          # needed for CadQuery functionality
                          freeglut3-dev \
                          # needed for CadQuery functionality
                          libosmesa6 \
                          # needed for CadQuery functionality
                          libosmesa6-dev \
                          # needed for CadQuery functionality
                          libgles2-mesa-dev \
                          # needed for Gmsh functionality
                          libxft2 \
                          # needed for gmsh
                          libxcursor-dev \
                          # needed for gmsh
                          libxinerama-dev \
                          # needed for pyvista
                          xvfb \
                          libgl1-mesa-glx \
                          libgl1-mesa-dri \
                          libosmesa6-dev \
                          libgl1-mesa-dev

RUN apt-get --yes install python3-pip python3-venv

# Enabling a venv within Docker is needed to avoid system wide installs
# https://pythonspeed.com/articles/activate-virtualenv-dockerfile/
ENV VIRTUAL_ENV=/opt/venv
RUN python3 -m venv $VIRTUAL_ENV
ENV PATH="$VIRTUAL_ENV/bin:$PATH"

RUN pip install --upgrade pip

# python packages from the neutronics workflow
RUN pip install neutronics_material_maker[density] \
                # coolprop \  failing to find 
                # stl_to_h5m \  failing to find 
                # remove_dagmc_tags \  failing to find 
                openmc-tally-unit-converter \
                regular_mesh_plotter \
                spectrum_plotter \
                # openmc_source_plotter \  failing to find 
                openmc_depletion_plotter \
                "openmc_data_downloader>=0.6.0" \
                retry \
                "openmc_data>=0.2.10" \
                openmc_plot \
                dagmc_geometry_slice_plotter \
                "cad_to_dagmc>=0.9.0" \
                "openmc-plasma-source>=0.3.1" \
                paramak --no-deps \
                mpmath \
                sympy \
                asteval \
                # pyvist \ # TODO add in pyvist but check it doesn't break the vtk/cad vis
                gmsh \
                pint \
                # 6.5.3-5 nbconvert is needed to avoid an error and that requires trixie debian OS
                # https://salsa.debian.org/python-team/packages/nbconvert/-/tags
                # https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=1068349
                lxml[html_clean]


RUN pip install git+https://github.com/svalinn/pydagmc

# Python libraries used in the workshop
RUN pip install plotly \
                # vtk \
                itkwidgets \
                pytest \
                ipywidgets \
                # cython is needed for moab and openmc, specific version tagged to avoid build errors
                "cython<3.0" \
                jupyterlab \
                jupyter-cadquery

# temporary wheels for moab hosted on github repo https://github.com/shimwell/wheels
RUN pip install https://github.com/shimwell/wheels/raw/refs/heads/main/moab/moab-wheels-ubuntu-latest/moab-5.5.1-cp311-cp311-manylinux_2_28_x86_64.whl  && \
    # temporary wheels for openmc hosted on github repo https://github.com/shimwell/wheels
    pip install https://github.com/shimwell/wheels/raw/refs/heads/main/openmc/openmc-0.15.3-cp311-cp311-manylinux_2_28_x86_64.whl && \
    # the order of these install appears to matter when it comes to jupyter vtk rendering
    pip install cadquery-vtk && \
    pip install git+https://github.com/CadQuery/cadquery.git@7cade87e68f2755fe7a121d797428c7b3d41b1be && \
    pip install pyvista panel trame-vtk trame ipywidgets itkwidgets


ENV DISPLAY=:99
ENV PORT=8888

# default username for mybinder
ARG NB_USER=jovyan
ARG NB_UID=1000
ENV USER=${NB_USER}
ENV NB_UID=${NB_UID}
ENV HOME=/home/${NB_USER}

# Create the user and set up home directory
RUN if ! id -u ${NB_UID} > /dev/null 2>&1; then \
        useradd --uid ${NB_UID} -m ${NB_USER}; \
    fi && \
    mkdir -p ${HOME} && \
    chown -R ${NB_UID}:${NB_UID} ${HOME} && \
    echo "${NB_USER}:x:${NB_UID}:${NB_UID}:${NB_USER},,,:${HOME}:/bin/bash" >> /etc/passwd

RUN mkdir -p ${HOME}/tasks && \
    chown -R ${NB_UID}:${NB_UID} ${HOME}/tasks && \
    chmod -R u+rwx ${HOME}/tasks

WORKDIR ${HOME}/tasks

RUN echo ${HOME}

RUN mkdir -p ~/nuclear_data

# Download and extract ENDF/b 8.0 chain file
RUN download_endf_chain -r b8.0 -d ~/nuclear_data -f chain-endf-b8.0.xml

# Download and extract ENDF/b 8.0 cross section files
RUN wget -q -O - https://anl.box.com/shared/static/uhbxlrx7hvxqw27psymfbhi7bx7s6u6a.xz | tar -C ~/nuclear_data -xJ  && \
    mv ~/nuclear_data/endfb-viii.0-hdf5/* ~/nuclear_data && \
    rm -rf ~/nuclear_data/endfb-viii.0-hdf5


# Download and extract WMP Library
RUN wget https://github.com/mit-crpg/WMP_Library/releases/download/v1.1/WMP_Library_v1.1.tar.gz -O ~/nuclear_data/WMP_Library_v1.1.tar.gz  && \
    tar -xzf ~/nuclear_data/WMP_Library_v1.1.tar.gz -C ~/nuclear_data && \
    rm ~/nuclear_data/WMP_Library_v1.1.tar.gz

# Copy over the local repository files
COPY tasks ${HOME}/tasks/

RUN mkdir -p ${HOME}/tasks/half-day-workshop

RUN cp ${HOME}/tasks/task_01_cross_sections/1_isotope_xs_plot.ipynb ${HOME}/tasks/half-day-workshop/task_01_isotope_xs_plot.ipynb  && \
    cp ${HOME}/tasks/task_01_cross_sections/2_element_xs_plot.ipynb ${HOME}/tasks/half-day-workshop/task_02_element_xs_plot.ipynb  && \
    cp ${HOME}/tasks/task_01_cross_sections/3_material_xs_plot.ipynb ${HOME}/tasks/half-day-workshop/task_03_material_xs_plot.ipynb  && \
    cp ${HOME}/tasks/task_02_making_materials/1_example_materials_from_isotopes.ipynb ${HOME}/tasks/half-day-workshop/task_04_example_materials_from_isotopes.ipynb && \
    cp ${HOME}/tasks/task_02_making_materials/2_example_materials_from_elements.ipynb ${HOME}/tasks/half-day-workshop/task_05_example_materials_from_elements.ipynb && \
    cp ${HOME}/tasks/task_03_making_CSG_geometry/1_simple_csg_geometry.ipynb ${HOME}/tasks/half-day-workshop/task_06_simple_csg_geometry.ipynb && \
    cp ${HOME}/tasks/task_04_make_sources/1_point_source_plots.ipynb ${HOME}/tasks/half-day-workshop/task_07_point_source_plots.ipynb && \
    cp ${HOME}/tasks/task_04_make_sources/2_ring_source.ipynb ${HOME}/tasks/half-day-workshop/task_08_ring_source.ipynb && \
    cp ${HOME}/tasks/task_04_make_sources/3_plasma_source_plots.ipynb ${HOME}/tasks/half-day-workshop/task_09_plasma_source_plots.ipynb && \
    cp ${HOME}/tasks/task_05_CSG_cell_tally_TBR/1_example_tritium_production.ipynb ${HOME}/tasks/half-day-workshop/task_10_example_tritium_production.ipynb && \
    cp ${HOME}/tasks/task_06_CSG_cell_tally_DPA/1_find_dpa.ipynb ${HOME}/tasks/half-day-workshop/task_11_find_dpa.ipynb && \
    cp ${HOME}/tasks/task_07_CSG_cell_tally_spectra/2_example_neutron_spectra_on_cell.ipynb ${HOME}/tasks/half-day-workshop/task_12_example_neutron_spectra_on_cell.ipynb && \
    cp ${HOME}/tasks/task_07_CSG_cell_tally_spectra/4_example_photon_spectra.ipynb ${HOME}/tasks/half-day-workshop/task_13_example_photon_spectra.ipynb && \
    cp ${HOME}/tasks/task_08_CSG_mesh_tally/1_example_2d_regular_mesh_tallies.ipynb ${HOME}/tasks/half-day-workshop/task_14_example_2d_regular_mesh_tallies.ipynb && \
    cp ${HOME}/tasks/task_14_variance_reduction/2_shielded_room_single_ww.ipynb ${HOME}/tasks/half-day-workshop/task_15_example_variane_shielded_room_ww_single_iteration.ipynb && \
    cp ${HOME}/tasks/task_14_variance_reduction/3_sphere_iterative_per_run_ww.ipynb ${HOME}/tasks/half-day-workshop/task_16_example_variance_reduction_sphere_ww_multiple_iterations.ipynb && \
    cp ${HOME}/tasks/task_10_activation_transmutation_depletion/3_full_pulse_schedule.ipynb ${HOME}/tasks/half-day-workshop/task_17_full_pulse_schedule.ipynb && \
    cp ${HOME}/tasks/task_21_design_task/1_optimal_design.ipynb ${HOME}/tasks/half-day-workshop/task_18_optimal_design.ipynb  && \
    cp ${HOME}/tasks/task_21_design_task/1_optimal_design_with_hints.ipynb ${HOME}/tasks/half-day-workshop/task_18_optimal_designs_with_hints.ipynb 

RUN chown -R ${NB_UID}:${NB_UID} /root
RUN chown -R ${NB_UID}:${NB_UID} ${HOME}/tasks

USER ${NB_USER}



# could switch to --ip='*'
CMD ["jupyter", "lab", "--notebook-dir=/home/jovyan/tasks", "--port=8888", "--no-browser", "--ip=0.0.0.0", "--allow-root"]

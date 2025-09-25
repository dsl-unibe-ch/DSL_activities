# Using conda on UBELIX

This is a complementary guide to the official UBELIX documentation about using conda environments on UBELIX. You can find documentation about conda [here](https://hpc-unibe-ch.github.io/software/installing/python/) and about using jupyter on UBELIX [here](https://hpc-unibe-ch.github.io/runjobs/ondemand/jupyter).

Here we try to give a complete overview and additionally indicate how to use containers to package environments.

In what follows, we create a custom environment and additionally make it available within the Jupyter instance that one can run via [UBELIX OnDemand](https://ondemand.hpc.unibe.ch/). Throughout this guide, we use the name `custom_env` for that environment and files related to it. You can use the name that you want, but we highly recommend to keep the *same name* in the various places where it is used.

## Plain conda env

The first solution is to just create a plain conda environment:
1. Use the installed Anaconda. First make conda available:
   ```
   module load Anaconda3
   eval "$(conda shell.bash hook)"
   ```
2. Then create your environment as usual. **Important**: include `ipykernel` in it. You can find an example of an environment file [here](https://raw.githubusercontent.com/dsl-unibe-ch/DSL_activities/refs/heads/main/assets/ubelix_conda/custom_env.yml)
3. Activate the environment and add it to your jupyter kernels. Follow [this](https://hpc-unibe-ch.github.io/runjobs/ondemand/jupyter/)
   ```
   conda activate custom_env
   python -m ipykernel install --user --name custom_env
   ```
4. Start a regular jupyter session on ondemand and you should see your environment as a kernel option.

## User container

Conda environments generate *a lot* of files and with multiple environments you will quickly reach your quota in number of files (1 million on UBELIX). We highly recommend to package conda environments within apptainer containers. Here's how to do it:

1. Get the `build.def` and `custom_env.yml` files that you can find in this repostitory as example [here](https://github.com/dsl-unibe-ch/DSL_activities/tree/main/assets/ubelix_conda)
2. Modify the environment file to your needs. You can rename the environment itself and the file but we recommend keeping the same name, here `custom_env`.
3. Adjust name of the file and environment in the in `build.def` file.
4. Upload the files to a folder in your home directory. In this example it is added to a `Support/ubelix_conda` folder.
5. Launch an UBELIX session on your terminal (either local or OnDemand). Launch an interactive session with enough RAM: `srun --mem=30G --time=3:00:00 --pty bash`
6. Move to the folder where you uploaded the files e.g. `cd ~/Support/ubelix_conda`
7. build the container. You can name the .sif file as you want: `apptainer build custom_env.sif build.def`
8. Start a regular jupyter session on UBELIX OnDemand.
9. Open a terminal within Jupyter and check if you have already kernels: `jupyter kernelspec list`. This should give you a list of locations. If the list is empty, you need to create the location `.local/share/jupyter/kernels/custom_env`
10. In that location create a file specifically called `kernel.json` that will contain the following information. Adjust the path to the sif file and the name of the environment.
    ```
    {
    "argv": [
        "singularity",
        "exec",
        "--cleanenv",
        "/storage/homefs/<username>/Support/ubelix_conda/custom_env.sif",
        "/opt/conda/envs/custom_env/bin/python",
        "-m",
        "ipykernel",
        "-f",
        "{connection_file}"
    ],
    "language": "python",
    "display_name": "custom_env"
    }
    ```
11. Refresh Jupyter and you should see your environment as a kernel option.


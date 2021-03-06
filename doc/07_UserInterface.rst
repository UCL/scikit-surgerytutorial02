.. highlight:: shell

.. _User_Interface:


Adding a User Interface
===============================================
Your library is sufficient as is, you have an implementation of an algorithm, which 
an interested user could download and use within their own Python application. However, 
it's nice to include a sample application, or some sort of user interface.  
A basic UI allows people to download and use your code directly and also see how 
your algorithm is meant to be used. The SciKit-Surgery Python template makes this
easy. 

Start by creating a new issue on WEISSlab, something like "Implement UI". And a new
git branch to match
::

   git checkout -b 2-implement-ui


We'll be modifying the code in the sksurgeryspherefitting/ui directory. 
Before we start, edit tests/pylintrc back to how it was, so our code gets properly tested.
::

   # Add files or directories to the blacklist. They should be base names, not
   # paths.
   ignore=CVS

Now edit sksurgeryspherefitting/ui/sksurgeryspherefitting_demo.py, so that 
it looks like:
::

  # coding=utf-8

  """Uses sphere fitting to fit to vtk model"""
  import vtk
  from sksurgeryvtk.models.vtk_surface_model import VTKSurfaceModel
  from sksurgeryspherefitting.algorithms import sphere_fitting

  def run_demo(model_file_name, output=""):
      """ Run the application """
      model = VTKSurfaceModel(model_file_name, [1., 0., 0.])
      x_values = model.get_points_as_numpy()[:, 0]
      y_values = model.get_points_as_numpy()[:, 1]
      z_values = model.get_points_as_numpy()[:, 2]

      initial_parameters = [0.0, 0.0, 0.0, 0.0]
      result = sphere_fitting.fit_sphere_least_squares(x_values,
                                                       y_values,
                                                       z_values,
                                                       initial_parameters)

      print("Result is {}".format(result))

      if output != "":

          sphere = vtk.vtkSphereSource()
          sphere.SetCenter(result[0][0], result[0][1], result[0][2])
          sphere.SetRadius(result[0][3])
          sphere.SetThetaResolution(60)
          sphere.SetPhiResolution(60)

          writer = vtk.vtkXMLPolyDataWriter()
          writer.SetFileName(output)
          writer.SetInputData(sphere.GetOutput())
          sphere.Update()
          writer.Write()

And edit sksurgeryspherefitting/ui/sksurgeryspherefitting_command_line.py:
::

  # coding=utf-8

  """Command line processing"""


  import argparse
  from sksurgeryspherefitting import __version__
  from sksurgeryspherefitting.ui.sksurgeryspherefitting_demo import run_demo


  def main(args=None):
      """Entry point for scikit-surgery-sphere-fitting application"""

      parser = argparse.ArgumentParser(
          description='scikit-surgery-sphere-fitting')

      ## ADD POSITIONAL ARGUMENTS
      parser.add_argument("model",
                          type=str,
                          help="Filename for vtk surface model")

      # ADD OPTINAL ARGUMENTS
      parser.add_argument("-o", "--output",
                          required=False,
                          type=str,
                          default="",
                          help="Write the fitted sphere to file"
                          )

      version_string = __version__
      friendly_version_string = version_string if version_string else 'unknown'
      parser.add_argument(
          "--version",
          action='version',
          version='scikit-surgery-sphere-fitting version '
          + friendly_version_string
          )

      args = parser.parse_args(args)

      run_demo(args.model, args.output)

We should also add a unit test to make sure that the demo program works, so create a file 
tests/test_sksurgeryspherefitting_demo.py and cut and paste this:
::

  # coding=utf-8

  """scikit-surgery-sphere-fitting tests"""

  from sksurgeryspherefitting.ui.sksurgeryspherefitting_demo import run_demo

  def test_fit_sphere_least_squares_demo():

      model_name = 'data/CT_Level_1.vtp'
      output_name = 'out_temp.vtp'

     run_demo (model_name, output_name)

Note that we need some testing data here. If you have a vtk surface file that you'd like to 
try fitting a sphere to you can subsitute it above. Other wise you can get one from `here`_
::

   mkdir data
   cd data
   wget https://weisslab.cs.ucl.ac.uk/StephenThompson/scikit-surgery-sphere-fitting/raw/master/data/CT_Level_1.vtp

Before you run tox again, we need to tell tox about the extra dependencies we've just added 
(`vtk`_, and `scikit-surgeryvtk`_)  so edit
requirements.txt, which should now look like:
::

   numpy
   scipy
   vtk
   scikit-surgeryvtk

Next we need to edit tests/pylintrc to help lint deal with python modules that use compiled libraries. 
Pylint can't see inside compiled libraries, so it needs help with "import vtk". So we add vtk to the 
"extension-pkg-whitelist" in pylintrc (line 32):
::

   extension-pkg-whitelist=numpy, vtk

If you run tox now, you should get all unit tests passing, and 100% test coverage. And if you're in the
project parent directory you should be able to run:
::

   python sksurgeryspherefitting data/CT_Level_1.vtp -o sphere.vtp

You'll see some output on the console, and if you have a vtk viewer you can load both models and see what 
you've done. Here's an example of a sphere fitted to a 3D ultrasound image of a fiducial sphere. 

The original US data:

.. figure:: https://github.com/UCL/scikit-surgerytutorial02/raw/master/doc/sphere.gif

and with a fitted sphere

.. figure:: https://github.com/UCL/scikit-surgerytutorial02/raw/master/doc/fitted_sphere.gif

If however you're using Python 2.7 on Windows tox will fail. Similarly, when you commit and push your changes, 
the continuous integration tests on WEISSLab will fail on windows. This because there is no python vtk package
available for Python 2.7 on Windows. We can edit tox.ini to fix this.

.. _`here`: https://gihub.com/thompson318/scikit-surgery-sphere-fitting/blob/master/data/CT_Level_1.vtp
.. _`vtk`: https://pypi.org/project/vtk/
.. _`scikit-surgeryvtk`: https://pypi.org/project/scikit-surgeryvtk/

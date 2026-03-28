# Pythonic Labjack library

The official python wrapper in `labjack-ljm` package is a literal translation of its native C API rather than *paraphrasing* it in pythonic way. It makes the wrapper horrible to comprehend and use. 

The goal of this project is to wrap the official wrapper one more time and use LabJacks with pythonic codes.

Let's comply with [PEP 8 Style Guide for Python Code](https://peps.python.org/pep-0008/) as much as possible. Particular focuses on this aspect are highlighted in [Code Style](#code-style) section.


## Stucture, usage & implementation guide

`labjack_device.py` and `_stream_in.py` would provide great examples for the below and more.

- Each LabJack device will be a object of `LabJackDevice` class in `labjack_device.py`. The constructor method and its arguments take whatever necessary to identify the device and connected to it.

    ```python
    # connect to LabJack
    lj_device = LabJackDevice(
        device_type=LabJackDeviceTypeEnum.T7,
        connection_type=LabJackConnectionTypeEnum.ETHERNET,
        device_identifier='192.168.1.128',
        )
    ```

- Each function of LabJack (e.g., stream_in) is to be implemented in its own internal script file (`StreamIn` class defined in `_stream_in.py`). Initiating the functions should be done by calling the corresponding `LabJackDevice` methods, and the results of the function will be returned an instance of the functions class.

    ```python
    # Example: triggered input stream
    stream_in = lj_device.stream_in(["AIN0", "AIN1"], 1, sampling_rate_Hz=50e3, do_trigger=True)
    
    # access to the result and stream settings
    print(stream_in) # __str__ has been defined
    total_nans = np.sum([np.isnan(value['V']).sum() for value in stream_in.records.values()])
    print(f"Recounting skipped total samples = {total_nans}")
    ```

- Each function class should implement a nice text representation of the measurement results via `__str__()` method.
- Each implemented feature should have script for test/demo in the script file's `if __name__ == "__main__":` block.


## Code style

This work put a massive effort on the code style for a good reason.

- Thorough and explicit type checking
- Very aware of [naming conventions](https://peps.python.org/pep-0008/#naming-conventions)
  - It is not about just naming but explicit designation of the variabls' rules and access scope! Some examples :
    - Typical variable: lowercas
    - Constant: UPPERCASE
    - Class: CapWords in general. Exceptional cases exist (e.g., naming for `Exception`s).
    - Access modifiers for variables and functions/methods, and script files: python interpreter doesn't care but still nice to be considerate.
      - public: (name)
      - protected: _(name) (single underscores at the beginning)
      - private: __(name) (double underscores at the beginning)
      - Particular focuses here:
        - Only the properties and methods that will be used by user (i.e., called from outside the top-level class) should be named as public.
        - Internal variables and methods should be named as `protected`.
        - `Eunm` is a class and thus has name with CapWords form.
    - Avoid using any forms that are reserved for other things in the style guide.
- Clear distinction between and implementation of properties and internal variables of classes.
  - (read-only) properties is set by defining their getters using `@property` decorator. Actual data should be stored in the corresponding internal variables named with `_my_obj` form.
  Setter (i.e., Write access to the public) is set by adding `@[variable_name].setter` decorator.
  
  ```python
  @property
  def my_obj(self): return self._my_obj
  @my_obj.setter
  def my_obj(self, value): self._my_obj = value
  ```

- Wrap finite, pre-defined argument values of functions/methods in `Enum` whenever possible. They are usually defined in `_ljm_aux.py` because it should act like global objects which can be acheived by putting `import _ljm_aux` in all the other script files.

    ```python
    # Example: Enum that lists the supported connection methods
    class LabJackConnectionTypeEnum(Enum):
        USB = ljm.constants.ctUSB
        ETHERNET = ljm.constants.ctETHERNET
        WIFI = ljm.constants.ctWIFI
    ```

    See [this section](#stucture-usage--implementation-guide) to for see example on how it is used.

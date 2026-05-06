Updated Readme.md and added notes to art.py

# Atomic Red Team Component

The Atomic Red Team (ART) component provides a means to connect scorch experiment configuration files to action Atomic Red Team attacks - using the ATT&CK framework. This provides a programmatic way to run attacks via experiments.

The goart executable:
https://github.com/activeshadow/go-atomicredteam

Reference Techniques, Index and Requirements:
https://github.com/redcanaryco/atomic-red-team/tree/master/atomics

```
type: art
exe:  phenix-scorch-component-art
```

It is assumed that the `goart` executable is available on the target VM in either the default path or the path specified in the configuration file. The component will verify the binary exists and is executable on the VM before attempting to run it.


## Executable Requirement

The `goart` binary must be injected, downloaded or preloaded onto each target VM. The examples below demonstrate injecting the binary/executable via the topology config.

topology.yml:

Windows VMs:

```yaml
injections:
  - src: /phenix/injects/${BRANCH_NAME}/art/goart.exe
    dst: /phenix/art/goart.exe
```

Linux VMs:

```yaml
injections:
  - src: /phenix/injects/${BRANCH_NAME}/art/goart
    dst: /phenix/art/goart
```

Scenario File (Scenario-Scorch.yml)
## Metadata Options

```yaml
metadata:
  framework: <local path to framework executable (e.g. goart)>
  technique: <technique ID>
  testName: <name of technique test>
  testIndex: <index of technique test>
  inputs: <map of input key/value pairs>
  technique: <technique ID> *REQUIRED
  testName: <name of technique test> *ONLY REQUIRED if testIndex is not present
  testIndex: <index of technique test> *ONLY REQUIRED if testName is not present
  goartPath: <path to goart executable on the VM> (optional) default: /phenix/art/goart or C:/phenix/art/goart.exe
  outputPath: <path to write goart output on the VM (optional, default: /phenix/art or /tmp)>
  execWaitSeconds: <seconds to wait after execution before retrieving results (optional, default: 5)>
  env: <map of environment variables>
  validator: <bash script to validate results>
  abortOnError: bool
  vms: <list of VM settings>
  vms: <list of VM settings> *REQUIRED
```

Only one of `testName` and `testIndex` needs to be provided. If both are
present, `testIndex` takes precedence. If neither are provided, index 0 will be
assumed.
present, `testIndex` takes precedence.

`goartPath` overrides the default goart binary path on the VM. Defaults to
`C:/phenix/art/goart.exe` for Windows and `/phenix/art/goart` for Linux.

`inputs` key/value pairs are used to fill in Atomic test variables.
`outputPath` overrides the directory where goart writes its JSON results on the
VM. Defaults to `/phenix/art` for Windows and `/tmp` for Linux. Note that for
Windows, this path must be accessible to the miniccc process.

`execWaitSeconds` controls how long the component waits after goart finishes
before attempting to retrieve the results file. Increase this for techniques
that have significant post-execution activity.

`env` key/value pairs are merged into the execution environment Atomic tests are
executed in (e.g. if the command a test is executing also expects some env
variables to be present).
executed in.

`validator` script should expect to be passed the hostname of the VM as the only
command line argument and the `.Executor.ExecutedCommand.results` portion of the
`vms` is a list of settings per VM to execute the test on.

## VM Settings

```yaml
vms:
  - hostname: <VM hostname> *REQUIRED
    inputs: <map of input key/value pairs> *REQUIRED as per technique information. Refer to Atomic Red Team Github.
    env: <map of environment variables>
    abortOnError: bool
```

`inputs` key/value pairs are used to fill in Atomic test variables for that
specific VM.

## Example Configuration

```yaml
components:
  - name: T1006
    type: art
    metadata:
      technique: T1006
      testIndex: 0
      execWaitSeconds: 10
      vms:
        - hostname: IT-ws10
          inputs:
            volume: "c:"
        - hostname: IT-ws11
          inputs:
            volume: "c:"
```

With optional overrides:

```yaml
components:
  - name: T1190
    type: art
    metadata:
      framework: /phenix/share/bin/goart
      technique: T1190
      testIndex: 0 # 0 is the default
      testIndex: 0
      goartPath: /custom/path/goart.exe
      outputPath: /custom/output
      execWaitSeconds: 15
      vms:
        - hostname: kali-inet
          abortOnError: false # false is the default
          abortOnError: false
          inputs:
            rhost: 172.29.0.25
            lhost: 1.2.3.4
      validator: |
        while read line; do
          if [[ "$line" ]] == *"Session 1 created"* ]]; then
          if [[ "$line" == *"Session 1 created"* ]]; then
            echo "T1190 ($1) -- Success"
            exit 0
          fi
        done

        echo "T1190 ($1) -- Failure" >&2
        exit 1
```
import os
import subprocess
import uuid
import time
import uuid

from box import Box

from phenix_apps.common.logger import logger

#This can be changed to reflect your directory structure on hosts
# This can be changed here to reflect your directory structure on hosts as a default.
# This is overwritten if a value is provided via goartPath
GOART_BASE = "/phenix/art"


class AtomicRedTeam(ComponentBase):
    def __init__(self):
        ComponentBase.__init__(self, "art")
            return "C:/phenix/art/goart.exe"
        return f"{GOART_BASE}/goart"
        

    def _tmp_path(self, os_type, filename):
        if os_type == "windows":
            return f"/phenix/art/{filename}"
                logger.info(f"results_file exists: {os.path.exists(results_file)}")
            except Exception as ex:
                raise RuntimeError(f"failed to get results file from {hostname}: {ex}") from ex
                raise RuntimeError(
                    f"failed to get results file from {hostname}: {ex}"
                ) from ex

            validator = self.metadata.get("validator", None)
            if validator:
                if proc.returncode != 0:
                    stderr = proc.stderr.decode()
                    logger.error(f"results validation failed: {stderr}" if stderr else "results validation failed")
                    logger.error(
                        f"results validation failed: {stderr}"
                        if stderr
                        else "results validation failed"
                    )
                    if abort_on_error:
                        raise RuntimeError("results validation failed")
                else:

if __name__ == "__main__":
    main()
    main()

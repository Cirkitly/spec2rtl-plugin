<purpose>
Initialize a new Hardware Design & Verification (HDV) project with standard directory structure and planning artifacts.
</purpose>

<process>

<step name="gather_info">
Ask user for project details if not provided in arguments:
- Project Name
- Technology (ASIC / FPGA / Soft IP)
- Target Device (e.g., Xilinx Ultrascale+, GF12nm)
- Language Preference (SystemVerilog / VHDL / Chisel)
</step>

<step name="scaffold_directories">
Create standard HDV directory structure:

```bash
mkdir -p rtl        # Design sources
mkdir -p tb         # Testbench sources
mkdir -p sim        # Simulation work directory
mkdir -p syn        # Synthesis work directory
mkdir -p docs       # Architecture/Spec documentation
mkdir -p scripts    # Build/Makefiles
mkdir -p .planning  # Spec2RTL Agent State
mkdir -p .planning/phases
mkdir -p .planning/milestones
```
</step>

<step name="init_planning_docs">
Instantiate core planning artifacts from templates.

**1. PROJECT.md**
Copy `templates/project.md` to `.planning/PROJECT.md`.
*Action:* Replace placeholders `{PROJECT_NAME}`, `{TECH_NODE}`.

**2. ROADMAP.md**
Copy `templates/roadmap.md` to `.planning/ROADMAP.md`.
*Action:* Set initial v1.0 milestone structure.

**3. STATE.md**
Copy `templates/state.md` to `.planning/STATE.md`.
*Action:* Initialize with "Phase 0: Planning".

**4. REQUIREMENTS.md**
Copy `templates/requirements.md` to `.planning/REQUIREMENTS.md`.
</step>

<step name="create_gitignore">
Create `.gitignore` tailored for EDA tools (Vivado, Quartus, Verilator).

```bash
cat <<EOF > .gitignore
# EDA Temp Files
*.log
*.jou
*.pb
*.wdb
*.vcd
*.fsdb
*.vpd
work/
INCA_libs/
xsim.dir/

# Synthesis/Implementation
syn/*.dcp
syn/*.bit
syn/*.reports
syn/rev/

# Simulation
sim/work
sim/transcript
sim/waves.vcd

# Editors
.vscode/
*.swp
EOF
```
</step>

<step name="init_git">
Initialize git repository if not already present.

```bash
git init
git add .
git commit -m "chore: initialize project structure"
```
</step>

<step name="create_makefile">
Create a basic Makefile in root.

```makefile
.PHONY: all clean sim lint

all: lint sim

lint:
	# Add lint command (e.g., verilator --lint-only)
	@echo "Linting..."

sim:
	# Add sim command
	@echo "Simulating..."

clean:
	rm -rf sim/work syn/output *.log
```
</step>

<step name="welcome">
Report success and suggest next step.

"Project {NAME} initialized.
Next: Define your architecture in `.planning/PROJECT.md` or start planning Phase 1:
`/s2r:plan-phase 1`"
</step>

</process>

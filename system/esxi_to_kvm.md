There're several files those are related to an ESXI image:

- `vmx` a template of the VM (can be compared to `.xml` for KVM) 
- `vmdk` a **real** image file (can be compared to `.qcow2` for KVM) 

if we have vmx and vmdk, the below command is recommended for image convertion

`virt-v2v -i vmx </path/to/.vmx> -o local -of qcow2 -os </path/to/des_dir>`

we will get both a template file and image(s) (depends on how many disk our VM uses)

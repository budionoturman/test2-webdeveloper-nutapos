<script setup lang="ts">
import { onMounted, ref } from 'vue';
import axios from 'axios'
import Dialog from 'primevue/dialog';
import ConfirmDialog from 'primevue/confirmdialog';
import { useConfirm } from "primevue/useconfirm";
import { useToast } from "primevue/usetoast";

const confirm = useConfirm();
const toast = useToast();

const apiBase = 'https://crudcrud.com/api/0e7b734289f541bf9cb47b22b6004d68/diskon'
const modalDiskon = ref(false)
const modalDiskonEdit = ref(false)
const d_Diskon = ref([]);
const dropdownToko = ref([
    { name: 'Toko Bagus', code: 'TB' },
    { name: 'https://crudcrud.com/api/0e7b734289f541bf9cb47b22b6004d68/diskon', code: 'API' },
]);
const dropdownValue = ref(dropdownToko.value[0]);
const input = ref({
    nama_toko: '',
    nama_diskon: '',
    persen_diskon: 0,
})

const input_edit = ref({
    _id: null,
    nama_toko: '',
    nama_diskon: '',
    persen_diskon: 0,
})

const filter = ref({
    nama_diskon: ''
})

const fetchDiskon = async () => {
    try {
        const res = await axios.get(apiBase)
        res.data.forEach((element, index) => {
            element.no = index + 1
        })

        if (filter.value.nama_diskon != '') {
            const keyword = filter.value.nama_diskon.toLowerCase()
            const filtered = res.data.filter(element => {
                return element.nama_diskon.toLowerCase().includes(keyword)
            })
            res.data = filtered
        }

        d_Diskon.value = res.data
    } catch (err) {
        console.error('Fetch error:', err)
    }

}

const modalTambahDiskon = () => {
    console.log('tambah diskon')
    input.value.nama_toko = dropdownValue.value.name;
    modalDiskon.value = true
}

const simpanDiskon = async () => {
    console.log('input', input.value)

    if (!input.value.nama_diskon || input.value.nama_diskon == '') {
        toast.add({ severity: 'error', summary: 'Alert', detail: 'Nama Diskon Wajib Diisi', life: 5000 });
        return
    }

    try {
        const res = await axios.post(apiBase, input.value);
        modalDiskon.value = false;
        fetchDiskon();
        clearInput();
        toast.add({ severity: 'info', summary: 'Alert', detail: 'berhasil simpan data', life: 5000 });
    } catch {
        toast.add({ severity: 'error', summary: 'Alert', detail: 'Gagal Simpan', life: 5000 });
    }
}

const modalEditDiskon = (data: any) => {
    input_edit.value._id = data._id;
    input_edit.value.nama_toko = data.nama_toko;
    input_edit.value.nama_diskon = data.nama_diskon;
    input_edit.value.persen_diskon = data.persen_diskon;

    modalDiskonEdit.value = true;
}

const editDiskon = async () => {
    const id = input_edit.value._id
    try {
        delete input_edit.value._id
        const res = await axios.put(apiBase + '/' + id, input_edit.value);
        modalDiskonEdit.value = false
        clearInput();
        fetchDiskon();
        toast.add({ severity: 'info', summary: 'Alert', detail: 'berhasil edit data', life: 5000 });
    } catch {
        toast.add({ severity: 'error', summary: 'Alert', detail: 'Gagal edit', life: 5000 });
    }
}

const hapusDiskon = async (data: any) => {
    confirm.require({
        message: 'Apakah anda yakin ingin menghapus diskon ' + data.nama_diskon,
        header: 'Danger Zone',
        icon: 'pi pi-info-circle',
        rejectLabel: 'Batalkan',
        rejectProps: {
            label: 'Batalkan',
            severity: 'secondary',
            outlined: true
        },
        acceptProps: {
            label: 'Hapus',
            severity: 'danger'
        },
        accept: () => {
            try {
                lanjutHapus(data)
            } catch {
                toast.add({ severity: 'error', summary: 'Alert', detail: 'Gagal hapus', life: 5000 });
            }
        },
        reject: () => {
            toast.add({ severity: 'error', summary: 'Batal', detail: 'Batal hapus', life: 5000 });
        }
    });
}

const lanjutHapus = async (data: any) => {
    try {
        const res = await axios.delete(apiBase + '/' + data._id)
        fetchDiskon();
        toast.add({ severity: 'info', summary: 'Alert', detail: 'Hapus berhasil', life: 5000 });
    } catch {
        toast.add({ severity: 'error', summary: 'Alert', detail: 'Gagal hapus', life: 5000 });
    }
}

const clearInput = () => {
    input.value = {
        nama_toko: '',
        nama_diskon: '',
        persen_diskon: 0,
    };

    input_edit.value = {
        _id: null,
        nama_toko: '',
        nama_diskon: '',
        persen_diskon: 0,
    }
}

onMounted(() => {
    // fetchDiskon()
});
</script>

<template>
    <ConfirmDialog></ConfirmDialog>
    <div class="card">
        <div class="grid grid-cols-12 gap-8">
            <div class="col-span-12 xl:col-span-6">
                <h1>Daftar Diskon</h1>
            </div>
        </div>
        <div class="grid grid-cols-12 gap-8">
            <div class="col-span-3 xl:col-span-9">
                <Select class="mr-2" v-model="dropdownValue" :options="dropdownToko" optionLabel="name"
                    placeholder="Select" @change="fetchDiskon()" />
                <InputText class="mr-2" v-model="filter.nama_diskon" type="text" placeholder="nama diskon" />
                <Button label="Cari" icon="pi pi-search" severity="success" @click="fetchDiskon" />
            </div>
            <div class="col-span-3 xl:col-span-3">
                <Button label="Tambah Diskon" severity="success" @click="modalTambahDiskon" />
            </div>
        </div>

        <div class="grid grid-cols-12 gap-8">
            <div class="col-span-12 xl:col-span-12">
                <div class="card">
                    <DataTable :value="d_Diskon" :rows="5" :paginator="true" responsiveLayout="scroll">
                        <Column field="no" header="No" :sortable="true" style="width: 1%"></Column>
                        <Column field="nama_toko" header="Nama Toko" :sortable="true" style="width: 20%"></Column>
                        <Column field="nama_diskon" header="Nama Diskon" :sortable="true" style="width: 20%"></Column>
                        <Column field="persen_diskon" header="Persen" :sortable="true" style="width: 20%"></Column>
                        <Column header="#" style="width: 20%">
                            <template #body="{ data }">
                                <Button icon="pi pi-pencil" severity="warn" rounded outlined
                                    @click="modalEditDiskon(data)" />
                                <Button icon="pi pi-trash" severity="danger" rounded outlined
                                    @click="hapusDiskon(data)" />
                            </template>
                        </Column>
                    </DataTable>
                </div>
            </div>
        </div>
    </div>

    <Dialog v-model:visible="modalDiskon" modal header="Tambah Diskon" :style="{ width: '50rem' }">
        <IftaLabel class="mb-2">
            <InputText id="nama diskon" type="text" v-model="input.nama_toko" />
            <label for="nama diskon">nama toko</label>
        </IftaLabel>
        <IftaLabel class="mb-2">
            <InputText id="nama diskon" type="text" v-model="input.nama_diskon" />
            <label for="nama diskon">nama diskon</label>
        </IftaLabel>
        <IftaLabel class="mb-2">
            <InputNumber id="nama diskon" type="text" v-model="input.persen_diskon" />
            <label for="nama diskon">persen diskon</label>
        </IftaLabel>
        <div class="flex justify-end gap-2">
            <Button type="button" label="Cancel" severity="secondary" @click="modalDiskon = false"></Button>
            <Button type="button" label="Save" @click="simpanDiskon"></Button>
        </div>
    </Dialog>

    <Dialog v-model:visible="modalDiskonEdit" modal header="Edit Diskon" :style="{ width: '50rem' }">
        <IftaLabel class="mb-2">
            <InputText id="nama diskon" type="text" v-model="input_edit.nama_toko" />
            <label for="nama diskon">nama toko</label>
        </IftaLabel>
        <IftaLabel class="mb-2">
            <InputText id="nama diskon" type="text" v-model="input_edit.nama_diskon" />
            <label for="nama diskon">nama diskon</label>
        </IftaLabel>
        <IftaLabel class="mb-2">
            <InputNumber id="nama diskon" type="text" v-model="input_edit.persen_diskon" />
            <label for="nama diskon">persen diskon</label>
        </IftaLabel>
        <div class="flex justify-end gap-2">
            <Button type="button" label="Cancel" severity="secondary" @click="modalDiskonEdit = false"></Button>
            <Button type="button" label="edit" @click="editDiskon"></Button>
        </div>
    </Dialog>
</template>

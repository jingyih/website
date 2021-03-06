---
title: Federation
content_template: templates/concept
weight: 80
---

{{% capture overview %}}

{{< deprecationfilewarning >}}
{{< include "federation-deprecation-warning-note.md" >}}
{{< /deprecationfilewarning >}}

Laman ini menjelaskan alasan dan cara penggunaan _federation_ untuk melakukan manajemen 
kluster Kubernetes.
{{% /capture %}}

{{% capture body %}}
## Kenapa _Federation_ ?

_Federation_ membuat proses manajemen kluster multipel menjadi lebih mudah. 
_Federation_ mencapai hal ini dengan cara menyediakan 2 buah fondasi:

  * Melakukan sinkronisasi _resource_ di seluruh kluster: _Federation_ 
    menyediakan kemampuan untuk melakukan sinkronisasi _resources_ pada _multiple_ 
    kluster. Sebagai contoh, kamu dapat memastikan _Deployment_ yang sama 
    tersedia pada kluster multipel. 
  * _Cross_ _cluster_ _Discovery_: _Federation_ menyediakan kemampuan untuk melakukan 
     konfigurasi otomatis server DNS dan _load balancer_ dari semua kluster. 
     Misalnya, kamu dapat memastikan bahwa sebuah VIP atau DNS global dapat digunakan 
     untuk mengakses _backend_ dari kluster multipel.

Beberapa penggunaan _federation_ adalah sebagai berikut:

* _High Availability_: Melakukan _load balance_ di seluruh kluster serta 
  melakukan konfigurasi otomatis server DNS dan _load balancer_, _federation_ 
  meminimalisasi dampak yang terjadi apabila terjadi kegagalan kluster.
* Mencegah _lock-in_ yang terjadi akibat penyedia layanan: Dengan cara mempermudah 
  proses migrasi antar kluster.


Manfaat _federation_ tidak akan terlalu kelihatan kecuali kamu memiliki beberapa kluster. 
Beberapa alasan kenapa kamu butuh beberapa kluster adalah:

* _Latency_ yang rendah: Memiliki kluster yang berada di _region_ yang berbeda 
  meminimalisasi _latency_ dengan cara menyajikan konten ke pengguna 
  berdasarkan _region_ yang paling dekat dengan pengguna tersebut.
* Isolasi _fault_: Akan lebih baik apabila kita memiliki beberapa kluster kecil 
  dibandingkan sebuah kluster besar untuk melakukan isolasi _fault_ (misalnya saja 
  kluster ini bisa saja berada di _availability_ zona dan penyedia layanan _cloud_ 
  yang berbeda).
* Skalabilitas: Terdapat batasan skalabilitas untuk sebuah kluster Kubernetes, 
  hal ini sebenarnya tidak menjadi masalah bagi sebagian besar pengguna. Untuk informasi 
  lebih lanjut kamu bisa membaca 
  [_Kubernetes Scaling_ dan Perencanaan Performa](https://git.k8s.io/community/sig-scalability/goals.md)).
* [_Hybrid cloud_](#hybrid-cloud-capabilities): Kamu dapat memiliki _multiple_ klsuter 
  pada penyedia layanan _cloud_ yang berbeda ataupun menggunakan _on-premsie_.

### Kekurangan

Meskipun terdapat banyak kelebihan dari penggunaan _federation_, 
terdapat beberapa kekurangan _federation_ yang dijabarkan sebagai berikut:

* Peningkatan _bandwidth_ dan biaya untuk jaringan: _control plane_ _federation_ bertugas mengawasi semua 
  kulster yang ada untuk menjamin _state_ yang ada saat ini sesuai dengan _state_ yang diinginkan. Hal ini dapat menyebabkan 
  peningkatan biaya jaringan apabila kluster yang ada dijalankan pada _region_ yang berbeda baik pada penyedia 
  layanan _cloud_ yang sama maupun berbeda.
* Berkurangnya isolasi antar kluster: Sebuah _bug_ yang ada pada _control plane_ _federation_ dapat 
  berdampak pada semua kluster. Hal ini dapat dihindari dengan cara mejaga logika yang ada pada _control plane_ _federation_ 
  seminimum mungkin.
* Kematangan: Proyek _federation_ ini tergolong baru dan belum cukup matang. 
  Tidak semua _resource_ yang ada tersedia dan masih banyak feature _alpha_. [_Issue_
  88](https://github.com/kubernetes/federation/issues/88) memberikan detail 
  isu-isu terkait sistem yang masih berusaha dicari solusinya.

### Kemampuan _Hybrid_ Penggunaan Layanan Penyedian _Cloud_

_Federation_ pada Kubernetes memungkinkan kluster untuk dijalankan 
pada penyedia layanan _cloud_ yang berbeda (misalnya Google Cloud, AWS), dan _on-premise_
(misalnya OpenStack). [Kubefed](/docs/tasks/federation/set-up-cluster-federation-kubefed/) 
adalah salah satu cara yang direkomendasikan untuk melakukan proses _deploy_ 
kluster _federation_.

Dengan demikian, [_resources_ API](#resources-api) yang kamu miliki 
dapat berada di kluster atau bahkan penyedia layanan _cloud_ yang berbeda.

## Mengaktifkan _Federation_

Untuk bisa melakukan _federation_ pada kluster yang berbeda, 
pertama kamu harus mengaktifkan _control plane_ _federation_.
Ikuti [petunjuk mengaktifkan _control plane_ _federation_](/docs/tutorials/federation/set-up-cluster-federation-kubefed/) 
untuk informasi lebih lanjut. 

## `Resources` API

Setelah kamu mengaktifkan _control plane_, kamu dapat menggunakan _resource_ API _federation_.
Berikut merupakan panduan yang akan menjelaskan masing-masing _resource_ secara mendetail:

* [Cluster](/docs/tasks/administer-federation/cluster/)
* [ConfigMap](/docs/tasks/administer-federation/configmap/)
* [DaemonSets](/docs/tasks/administer-federation/daemonset/)
* [Deployment](/docs/tasks/administer-federation/deployment/)
* [Events](/docs/tasks/administer-federation/events/)
* [Hpa](/docs/tasks/administer-federation/hpa/)
* [Ingress](/docs/tasks/administer-federation/ingress/)
* [Jobs](/docs/tasks/administer-federation/job/)
* [Namespaces](/docs/tasks/administer-federation/namespaces/)
* [ReplicaSets](/docs/tasks/administer-federation/replicaset/)
* [Secrets](/docs/tasks/administer-federation/secret/)
* [Services](/docs/concepts/cluster-administration/federation-service-discovery/)


[Referensi Dokumentasi API](/docs/reference/federation/) memberikan semua daftar 
_resources_ yang disediakan _apiserver_ _federation_.

## Penghapusan Berantai

Kubernetes versi 1.6 menyediakan mekanisme penghapusan berantai 
untuk _resource_ yang ada pada _federation_. Dengan penghapusan berantai, 
ketika kamu menghapus sebuah _resource_ dari _control plane_ _federation_, 
kamu juga akan menghapus segala _resource_ tersebut pada semua kluster yang ada.

Mekanisme penghapusan berantai ini tidak diaktifkan secara _default_ 
ketika menggunakan REST API. Untuk mengaktifkannya, ubah nilai dari opsi 
`DeleteOptions.orphanDependents=false` ketika kamu menghapus sebuah _resource_ 
dari _control plane_ _federation_ dengan menggunakan REST API. 
Penggunaan `kubectl delete`mengaktifkan penhapusan berantai secara _default_. 
Kamu dapat menonaktifkannya dengan menggunakan `kubectl delete --cascade=false`

Catatan: Kubernetes versi 1.5 menyediakan penghapusan berantai 
untuk sebagian _resource_ _federation_.

## Cakupan dari Sebuah Kluster

Pada penyedia IaaS seperti Google Compute Engine atau Amazon Web Services, sebuah VM ada di dalam 
[zona](https://cloud.google.com/compute/docs/zones) atau [_availability
zone_](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-regions-availability-zones.html).
Kami menyarankan agar semua VM pada kluster Kubernetes berada pada _availability_ zona yang sama, karena: 

  - dibandingkan dengan sebuah kluster global Kubernetes, terdapat lebih sedikit _single-points of failure_.
  - dibandingkan dengan sebuah kluster yang tersebar pada _availability zone_ yang mungkin berbeda, akan lebih mudah untuk merencanakan properti _availability_ dari sebuah 
    kluster yang berada pada satu zona.
  - ketika pengembang Kubernetes mendesain sistem (misalnya, memperkirakan _latency_, _bandwidth_, atau 
    _failure_ yang mungkin terjadi) pengembang tersebut memperkirakan semua mesin akan berada pada sebuah _data center_ yang sama, atau setidaknya masih terdapat pada satu wilayah.

Sangat direkomendasikan untuk menjalankan sedikit kluster dengan lebih banyak VM pada setiap _availability_ zona; 
meskipun begitu hal ini tidak menutup kemungkinan untuk menjalankan kluster multipel 
pada setiap _availability_ zona.

Alasan kenapa menjalankan lebih sedikit kluster pada setiap _availability_ zona lebih dianjurkan:

  - meningkatkan _bin packing_ _Pod_ pada beberapa kasus dimana terdapat lebih banyak _node_ dalam sebuah kluster (mengurangi terjadinya _fragmentation_ _resource_).
  - mengurangi _overhead_ operasional (meskipun keuntungan ini akan berkurang seiring bertambah matangnya proses dan _tooling_ operasional).
  - mengurangi biaya _resource_ tetap per kluster, misalnya VM _apiserver_.

Alasan untuk memiliki kluster multipel:

  - _policy_ kemananan yang ketat membutuhkan isolasi antar _work_ _class_ (baca Partisi Kluster di bawah).
  - melakukan penerapan Kubernetes dan/atau perangkat lunak lain yang versi baru ke salah satu kluster.

## Memilih jumlah kluster yang tepat

Pemilihan jumlah kluster yang tepat merupakan pilihan yang relatif statis, dan hanya akan ditinjau kembali sewaktu-waktu. 
Sebaliknya, jumlah _node_ dan _pod_ dalam suatu _service_ dapat berubah secara cepat seiring bertambahnya _workload_.

Untuk memilih jumlah kluster, pertama, pilih _region_ yang memiliki _latency_ yang masih dapat dimaklumi untuk semua pengguna aplikasi kamu 
(jika kamu menggunakan _Content Distribution Network_, kebutuhan informasi nilai _latency_ CDN tidak perlu diperhatikan). 
Masalah legal juga perlu diperhitungkan. Misalnya sebuah perusahaan dengan pelanggan global bisa jadi memilih kluster di _region_ 
US, EU, AP, dan SA. Jumlah _region_ ini dimisalkan dengan `R`.

Kedua, pilih berapa banyak kluster yang bisa jadi _unavailable_ secara bersamaan tanpa membuat _service_ menjadi _unavailable_. 
Misalkan jumlah kluster _unavailable_ ini sebagai `U`. Jika kamu tidak yakin, maka 1 merupakan pilihan yang tergolong 
dapat diterima.

Jika aplikasimu memungkinkan trafik untuk di-_load balance_ ke _region_ mana saja ketika terjadi _failure_ pada kluster, 
maka kamu setidaknya membutuhkan nilai yang lebih banyak dari jumlah `R` atau `U + 1` kluster. Jika tidak (misalnya, kamu 
ingin menjamin stabilnya _latency_ ketika terjadi _failure_ pada kluster) maka kamu membutuhkan `R * (U + 1)` kluster 
(`U + 1` di setiap _region_ yang ada pada `R`). Pada kasus lain, cobalah untuk menerapkan satu kluster 
 pada zona yang berbeda.

Terakhir, jika kluster yang kamu miliki membutuhkan jumlah _node_ yang melebihi nilai yang direkomendasikan untuk sebuah kluster Kubernetes, 
maka kamu membutuhkan lebih banyak kluster. Kubernetes v1.3 mampu menangani hingga 1000 node untuk setiap kluster. Kubernetes v1.8 
mampu menangani hingga 5000 node untuk tiap kluster. Baca [Membangun Kluster Besar](/docs/setup/cluster-large/) untuk petunjuk lebih lanjut.

{{% /capture %}}

{{% capture whatsnext %}}
* Pelajari lebih lanjut tentang [proposal
  _Federation_](https://github.com/kubernetes/community/blob/{{< param "githubbranch" >}}/contributors/design-proposals/multicluster/federation.md).
* Baca [petunjuk pengaktifan](/docs/tutorials/federation/set-up-cluster-federation-kubefed/) kluster _federation_.
* Lihat [seminar tentang _federation_ pada Kubecon2016](https://www.youtube.com/watch?v=pq9lbkmxpS8)
* Lihat [_update_ _federation_ pada Kubecon2017 Eropa](https://www.youtube.com/watch?v=kwOvOLnFYck)
* Lihat [_update_ _sig-multicluster_ pada Kubecon2018 Eropa](https://www.youtube.com/watch?v=vGZo5DaThQU)
* Lihat [presentasi prototipe _Federation-v2_ pada Kubecon2018 Eropa](https://youtu.be/q27rbaX5Jis?t=7m20s)
* Lihat [petunjuk penggunaan _Federation-v2_](https://github.com/kubernetes-sigs/federation-v2/blob/master/docs/userguide.md)
{{% /capture %}}

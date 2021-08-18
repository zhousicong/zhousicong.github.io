---
title: s3-exporter实现
description: '对CEPH S3的bucket容量监控,依赖于rados-admin命令的执行'
categories:
  - CEPH
  - S3
tags:
  - 监控
abbrlink: f7f721f2
date: 2021-08-17 15:27:14
---
```go
 package main

import (
	"context"
	"encoding/json"
	"flag"
	"fmt"
	"log"
	"net/http"
	"os/exec"
	"strconv"
	"sync"
	"time"

	"github.com/prometheus/client_golang/prometheus"
	"github.com/prometheus/client_golang/prometheus/promhttp"
)

var (
	rootDir string
	depth   int
	port    int
)

type BucketUsage struct {
	Bucket string `json:"bucket"`
	Owner  string `json:"owner"`
	Usage  struct {
		RGWMain struct {
			Size         float64 `json:"size"`
			SizeActual   float64 `json:"size_actual"`
			SizeUtilized float64 `json:"size_utilized"`
			NumObjects   float64 `json:"num_objects"`
		} `json:"rgw.main"`
	} `json:"usage"`
	BucketQuota struct {
		Enable     bool    `json:"enabled"`
		MaxSize    float64 `json:"max_size"`
		MaxSizeKB  float64 `json:"max_size_kb"`
		MaxObjects float64 `json:"max_objects"`
	} `json:"bucket_quota"`
}

const namespace string = "unionstor_radosgw_usage"

type Collector struct {
	mu                      sync.Mutex
	BucketSizeBytes         *prometheus.GaugeVec
	BucketSizeUtilizedBytes *prometheus.GaugeVec
	BucketNumsObjects       *prometheus.GaugeVec
	BucketQuotaEnable       *prometheus.GaugeVec
	BucketQuotaMaxSize      *prometheus.GaugeVec
	BucketQuotaMaxSizeBytes *prometheus.GaugeVec
	BucketQuotaMaxObjects   *prometheus.GaugeVec
}

func exec_cmd(command string, timeout int) ([]byte, error) {
	ctx, cancel := context.WithTimeout(context.Background(), time.Second*time.Duration(timeout))
	defer cancel()

	cmd := exec.CommandContext(ctx, "bash", "-c", command)
	out, err := cmd.CombinedOutput()
	return out, err
}

func NewCollector() *Collector {
	return &Collector{
		BucketSizeBytes: prometheus.NewGaugeVec(prometheus.GaugeOpts{
			Namespace: namespace,
			Name:      "bucket_size_bytes",
			Help:      "bucket_size_bytes",
		}, []string{"bucket", "owner"}),
		BucketSizeUtilizedBytes: prometheus.NewGaugeVec(prometheus.GaugeOpts{
			Namespace: namespace,
			Name:      "bucket_size_utilized_bytes",
			Help:      "bucket_size_utilized_bytes",
		}, []string{"bucket", "owner"}),
		BucketNumsObjects: prometheus.NewGaugeVec(prometheus.GaugeOpts{
			Namespace: namespace,
			Name:      "bucket_num_objects",
			Help:      "bucket_num_objects",
		}, []string{"bucket", "owner"}),
		BucketQuotaEnable: prometheus.NewGaugeVec(prometheus.GaugeOpts{
			Namespace: namespace,
			Name:      "bucket_quota_enable",
			Help:      "bucket_quota_enable",
		}, []string{"bucket", "owner"}),
		BucketQuotaMaxSize: prometheus.NewGaugeVec(prometheus.GaugeOpts{
			Namespace: namespace,
			Name:      "bucket_quota_max_size",
			Help:      "bucket_quota_max_size",
		}, []string{"bucket", "owner"}),
		BucketQuotaMaxSizeBytes: prometheus.NewGaugeVec(prometheus.GaugeOpts{
			Namespace: namespace,
			Name:      "bucket_quota_max_size_bytes",
			Help:      "bucket_quota_max_size_bytes",
		}, []string{"bucket", "owner"}),
		BucketQuotaMaxObjects: prometheus.NewGaugeVec(prometheus.GaugeOpts{
			Namespace: namespace,
			Name:      "bucket_quota_max_objects",
			Help:      "bucket_quota_max_objects",
		}, []string{"bucket", "owner"}),
	}
}
func (c *Collector) collectorList() []prometheus.Collector {
	return []prometheus.Collector{
		c.BucketSizeBytes,
		c.BucketSizeUtilizedBytes,
		c.BucketNumsObjects,
		c.BucketQuotaEnable,
		c.BucketQuotaMaxSize,
		c.BucketQuotaMaxSizeBytes,
		c.BucketQuotaMaxObjects,
	}
}

func (c *Collector) Describe(ch chan<- *prometheus.Desc) {
	for _, metric := range c.collectorList() {
		metric.Describe(ch)
	}
}

func (c *Collector) Collect(ch chan<- prometheus.Metric) {
	c.mu.Lock()
	defer c.mu.Unlock()
	if err := c.collect(); err != nil {
		log.Println(err.Error())
		return
	}
	for _, metric := range c.collectorList() {
		metric.Collect(ch)
	}
}

func (c *Collector) collect() error {
	cmd := "radosgw-admin bucket stats"
	cmdRet, err := exec_cmd(cmd, 30)
	if err != nil {
		fmt.Println("exec bucket stats error:", err)
		return err
	}
	bucketsUsage := []BucketUsage{}
	err = json.Unmarshal(cmdRet, &bucketsUsage)
	if err != nil {
		fmt.Println("parse cmd returen error:", err)
		return err
	}
	c.BucketSizeBytes.Reset()
	c.BucketSizeUtilizedBytes.Reset()
	c.BucketNumsObjects.Reset()
	c.BucketQuotaEnable.Reset()
	c.BucketQuotaMaxSize.Reset()
	c.BucketQuotaMaxSizeBytes.Reset()
	c.BucketQuotaMaxObjects.Reset()
	for _, bucketUsage := range bucketsUsage {
		bucketName := bucketUsage.Bucket
		bucketOwner := bucketUsage.Owner
		bucketSizeBytes := bucketUsage.Usage.RGWMain.SizeActual
		bucketSizeUtilizedBytes := bucketUsage.Usage.RGWMain.SizeUtilized
		bucketObjects := bucketUsage.Usage.RGWMain.NumObjects
		var bucketQuotaEnable float64 = 0
		if bucketUsage.BucketQuota.Enable {
			bucketQuotaEnable = 1
		}
		bucketQuotaMaxSize := bucketUsage.BucketQuota.MaxSize
		bucketQuotaMaxSizeBytes := bucketUsage.BucketQuota.MaxSizeKB
		bucketQuotaMaxObjects := bucketUsage.BucketQuota.MaxObjects
		c.BucketSizeBytes.WithLabelValues(bucketName, bucketOwner).Set(bucketSizeBytes)
		c.BucketSizeUtilizedBytes.WithLabelValues(bucketName, bucketOwner).Set(bucketSizeUtilizedBytes)
		c.BucketNumsObjects.WithLabelValues(bucketName, bucketOwner).Set(bucketObjects)
		c.BucketQuotaEnable.WithLabelValues(bucketName, bucketOwner).Set(bucketQuotaEnable)
		c.BucketQuotaMaxSize.WithLabelValues(bucketName, bucketOwner).Set(bucketQuotaMaxSize)
		c.BucketQuotaMaxSizeBytes.WithLabelValues(bucketName, bucketOwner).Set(bucketQuotaMaxSizeBytes)
		c.BucketQuotaMaxObjects.WithLabelValues(bucketName, bucketOwner).Set(bucketQuotaMaxObjects)

	}

	return nil
}

func main() {
	flag.StringVar(&rootDir, "path", "/", "A cephfs path")
	flag.IntVar(&depth, "depth", 2, "default depth")
	flag.IntVar(&port, "port", 8080, "default exporter port")
	flag.Parse()
	prometheus.MustRegister(NewCollector())
	http.Handle("/metrics", promhttp.Handler())
	log.Fatal(http.ListenAndServe(":"+strconv.Itoa(port), nil))
}
```
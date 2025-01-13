# Repliforge AWS

This script runs reproduicibility test for FreeBSD images in AWS. Provided an
AMI(Amazon Machine Images) id it will determine the GIT Hash and Branch from
which the FreeBSD image was built and will try to rebuild an identical image.
The diffs are tested with [diffoscope](https://diffoscope.org). diffoscope is
by default a single threaded process and takes a substantial amount of time
ranging from 8-12 hours.

## Usage

The main script is `repliforge`. It takes the following arguments:

- `-a` which requires the AMI ID of the image. Please be mindful of the
    combination of the AMI ID and the REGION.
- `-s` security group configured in AWS.
- `-k` name of the ssh keypair id configured in aws
- `-t` instance type is the type of the instance like `c6i.4xlarge`
- `-r` defines the region. Unless specified it tries to read from the aws cli
    configurations or the presence of the environment variables like
    `AWS_REGION` or `AWS_DEFAULT_REGION`. If everything fails then the program
    will terminate with error.
- `-d` is the name of the instance; unless specified it takes the form of
    `FreeBSD-<INSTANCE_TYPE>`.
- `-v` defines the size of the EBS(Elastic Block Storage) size, default is 44GB

## Process

During the initialization the script checks that `aws` was configured properly
to initiate creating a new instance. This is done by checking if `aws` cli is
present. Depending on various operating system the package can be installes.

In the next step it checks for the presence of the environment variables
`AWS_ACCESS_KEY_ID` or `AWS_SECRET_ACCESS_KEY`. If not present it checks for
the presence of the file `~/.aws/credentials` and `~/aws/config` and if found
moves to the next stept to check if `aws` cli can execute some simple api
calls.

On the next step it checks for the configured regions. Mainly the presence of
the environment variables like `AWS_REGION` or `AWS_DEFAULT_REGION`. As a
fail through it checks the presence of the `~/.aws/config` to get the region
and use it. In all other cases `-r` argument must be passed.

If everything runs properly there will be an output file named
`freebsd-<AMIID>.html` in the home directory. In case of failure there will be
a file named `output-<AMIID>.log` . The script continuously tries to check the
system if the process has finished. And at the end of the process terminates
the AWS instance.

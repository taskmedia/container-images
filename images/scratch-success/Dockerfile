FROM golang as build
COPY main.go main.go
RUN go build -o main main.go
RUN echo "scratchuser:x:1001:1001::/home/scratchuser:/bin/sh" > passwd

FROM scratch
COPY --from=build /go/main /main
COPY --from=build /go/passwd /etc/passwd
USER 1001
CMD [ "/main" ]

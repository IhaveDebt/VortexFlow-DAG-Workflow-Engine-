//
//  VortexFlow.swift
//

import Foundation

class FlowTask {
    let id: String
    let execute: () -> Void
    var dependencies: [FlowTask] = []
    
    init(id: String, execute: @escaping () -> Void) {
        self.id = id
        self.execute = execute
    }
}

class VortexFlow {
    
    private let queue = DispatchQueue(label: "vortex.engine", attributes: .concurrent)
    
    func run(tasks: [FlowTask]) {
        for task in tasks {
            queue.async {
                self.runTask(task)
            }
        }
    }
    
    private func runTask(_ task: FlowTask) {
        for dependency in task.dependencies {
            dependency.execute()
        }
        
        print("Executing task:", task.id)
        task.execute()
    }
}

let build = FlowTask(id: "build") { print("Compile source…") }
let test = FlowTask(id: "test") { print("Running tests…") }
let package = FlowTask(id: "package") { print("Packaging…") }

test.dependencies = [build]
package.dependencies = [test]

let engine = VortexFlow()
engine.run(tasks: [build, test, package])

RunLoop.main.run()
